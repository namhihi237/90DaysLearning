# Index

## Postgres
### Chuẩn bị

1. Tạo dummy data với table `users`
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    username VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL,
    avatar TEXT,
    password VARCHAR(255) NOT NULL,
    birthdate DATE,
    registered_at TIMESTAMP,
    age INT(2),
    status VARCHAR(50) NOT NULL DEFAULT 'active'
);
```

<details>
<summary>2. Script tạo 5M users</summary>


```js
const { Client } = require('pg');
const { faker } = require('@faker-js/faker');

const client = new Client({
  user: 'postgres',
  host: 'localhost',
  database: 'test-large',
  password: '',
  port: 5432,
});

const BATCH_SIZE = 10000;
const TOTAL_USERS = 5000000;

const statuses = ['active', 'inactive', 'banned'];
async function createRandomUser() {
  return {
    id: faker.string.uuid(),
    username: faker.internet.userName(),
    email: faker.internet.email(),
    avatar: faker.image.avatar(),
    password: faker.internet.password(),
    birthdate: faker.date.birthdate(),
    registered_at: faker.date.past(),
    status: faker.helpers.arrayElement(statuses),
    age: faker.number.int({ min: 1, max: 99 })
  };
}

async function insertUsers(batch) {
  const values = batch.map(user =>
    `('${user.id}', '${user.username}', '${user.email}', '${user.avatar}', '${user.password}', '${user.birthdate.toISOString()}', '${user.registered_at.toISOString()}', '${user.status}')`
  ).join(',');

  const query = `INSERT INTO users (id, username, email, avatar, password, birthdate, registered_at, status) VALUES ${values}`;

  try {
    await client.query(query);
  } catch (error) {
    console.error('Error inserting batch:', error);
  }
}

async function main() {
  try {
    await client.connect();

    for (let i = 0; i < TOTAL_USERS; i += BATCH_SIZE) {
      const batch = [];
      for (let j = 0; j < BATCH_SIZE; j++) {
        batch.push(await createRandomUser());
      }
      await insertUsers(batch);
      console.log(`Inserted batch ${i / BATCH_SIZE + 1}`);
    }

    await client.end();
  } catch (error) {
    console.log(error);

  }
}

main().catch(err => console.error('Error in main:', err));

```
</details>

### Các loại index
Trong postgres chúng ta có các loại index như sau, và mỗi index sử dụng thuật toán khác nhau
- B-Tree
- Hash
- GiST
- SP-GiST
- GIN
- BRIN

Khi đánh index sử dụng câu lệnh `CREATE INDEX`, mặc định nó sử dụng `B-TREE` index.


Nếu chúng ta muốn chọn index type:
```sql
CREATE INDEX name ON table USING HASH (column);
```

Postgres sẽ xem xem xét sử dụng B-Tree bất cứ khi nào một cột được lập chỉ mục có liên quan đến việc so sánh sử dụng một trong các toán tử
`<   <=   =   >=   >`

Ngoài ra còn có `LIKE` and `~` với pattern bắt đầu với một chuỗi string không đổi như `foo%` hoặc `^foo`.

Đối với `%bar` nó sẽ không hỗ trợ bởi B-Tree index


### Multicolumn Indexs
Một index có thể được xác định bởi nhiều hơn 1 cột

Nếu bạn thường xuyên sử dụng query như thế này:
```sql
select * from users where status = 'active' and age > 10;
```

Chúng ta có thể tiệp cận đánh index cho 2 cột status và age. Nhưng việc thứ tự của việc đánh index sẽ ảnh hưởng đến hiệu suất

```sql
CREATE INDEX idx_status_age ON users (status, age);
```

```sql
CREATE INDEX idx_status_age ON users (age, status);
```

Vậy trong trường hợp này đâu là cách chúng ta nên sử dụng? Để việc đánh index hiệu quả chúng ta có tư duy sau, đối với status chúng ta sẽ có một số status ở ví dụ trên là `3` nhưng đối với tuổi chúng ta sẽ có độ dàn trải từ `1 -> 99` Vậy nên để index được hiểu quả việc sử dụng lọc status trước sẽ loại bỏ được phần lớn rows. Vì thế chúng ta nên sử dụng cách thứ 1.

A multicolumn B-tree hiệu quả khi có các ràng buộc trên các cột đầu tiên (leftmost). Index sẽ hoạt động hiệu quả nhấ khi query chỉ định các điều kiện trên các cột chính này.

Ví dụ chúng ta có truy vấn như sau
```sql
create index abc on test(a,b,c);

select * from test where a = 5 and b  > 43 and c < 23;
```
Trong trường hợp này chỉ mục sẽ hoạt động hiệu quả vì điều kiện truy vấn phù hợp với cấu trúc của index.
- Ràng buộc trên `a` giúp index tìm ra các row có a = 5
- Ràng buộc trên `b` điều kiện  `b > 43` sẽ giúp index lọc các hàng có b > 43 và vì  b là cột sau a trong index nên nó sẽ tiếp tục quét các mục có  b > 43 sau khi tìm thấy các mục có a = 5
- Ràng buộc trên `c` với điều kiện c < 23 sẽ giúp loại bỏ các hàng có c >= 23. Vì c là cột cuối cùng của index, nên index sẽ quét qua các chỉ mục với a = 5, b > 43 và chỉ lấy c < 23

Chúng ta có ví dụ về chỉ mục không hiệu quả
```sql
create index abc on test(a,b,c);

select * from test where c = 5 and b  > 43 and a < 23;
```

Trong trường hợp này index sẽ không hiệu quả vì query không khớp với cấu trúc index.
- Khó định vị c vì c không phải cột đầu tiên trong index, chỉ mục không thể nhanh chóng xác định c = 5 mà phải quét qua toàn bộ bảng.
- Ràng buộc trên b, chỉ mục không thể làm giảm phạm vi scan nếu không có điều kiện phụ thuộc vào a.

Trong trường hợp này nó sẽ ưu tiên scan toàn bộ bảng thay vì sử dụng index.

### Phân tích kế hoạch thực thi
Sử dụng: EXPLAIN ANALYZE

Ví dụ
```sql
EXPLAIN ANALYZE select * from users where status = 'active' and age > 10;
```

Chưa đánh index
```
Gather  (cost=1000.00..158319.44 rows=1 width=164) (actual time=1431.147..1432.106 rows=0 loops=1)"
  Workers Planned: 2"
  Workers Launched: 2"
  ->  Parallel Seq Scan on users3  (cost=0.00..157319.34 rows=1 width=164) (actual time=1426.561..1426.562 rows=0 loops=3)"
        Filter: ((age > 10) AND ((status)::text = 'active'::text))"
        Rows Removed by Filter: 1666667"
Planning Time: 0.081 ms"
Execution Time: 1433.136 ms"
```

Ở đây chúng ta thấy nó sử dụng 2 worker để scan tòn bộ table

Đánh index
```sql
create index status_age on users(status, age);
```

```
Index Scan using status_age on users3  (cost=0.43..8.22 rows=1 width=164) (actual time=0.030..0.030 rows=0 loops=1)
  Index Cond: (((status)::text = 'active'::text) AND (age > 10))
Planning Time: 0.397 ms
Execution Time: 0.047 ms
```

Sau khi đánh index, thì nó đã sử dụng Index scan.

Dễ thấy thời gian giảm từ `1433ms -> 0,047ms`