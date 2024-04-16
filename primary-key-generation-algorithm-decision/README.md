## Overview
Primary Key 생성 알고리즘을 결정하는 것은 시스템에 광범위하게 영향을 끼치는 요소 중 하나입니다. 데이터의 무결성과 쿼리의 효율성은 물론, 나아가 시스템 전체의 확장성에까지 중대한 영향을 미칩니다. 이처럼 핵심적인 요소로서의 Primary Key는, 그 생성 알고리즘을 올바르게 결정하지 못하면 시스템의 성공 자체를 위협할 수 있습니다.

## What is a Primary Key?
Primary Key는 Table에서 Row를 고유하게 식별하는 Column을 말합니다. Primary Key는 테이블 내에서 중복된 값이나 NULL값을 가질 수 없습니다.
아래 Invariant를 만족해야합니다.
```ts
for (const row of table) {
  const count = table.count({
    where: {
      primaryKey: row.primaryKey
    }
  });

  if (count !== 1) {
    throw new Error('Primary Key must be unique in the table.');
  }
}
```
## Clustered Index
In a database with a clustered index, the rows are physically stored on the disk in the same order as the index values. Therefore, each table can only have one clustered index. While many relational database management systems (RDBMS) automatically create a clustered index on the primary key column by default, this is not a universal rule. The structure of a clustered index organizes the data in such a way that it reflects the indexed order, facilitating efficient data retrieval and storage operations.
```ts
const ClusteredIndex = [
  {id: 1, page: 100},
  {id: 5, page: 101},
  {id: 9, page: 102},
];

const PhysicalDisk = {
  pageSize: 4,
  pages: {
    100: [
      {id: 1, name: 'Anna',   phone: '010-2320-1234'},
      {id: 2, name: 'Lee',    phone: '010-1421-4567'},
      {id: 3, name: 'Sarah',  phone: '010-9981-7890'},
      {id: 4, name: 'Daniel', phone: '010-5555-4321'},  
    ],
    101: [
      {id: 5, name: 'Erica',  phone: '010-1342-2345'},
      {id: 6, name: 'Oscar',  phone: '010-8765-5432'},
      {id: 7, name: 'Julie',  phone: '010-2222-1111'},
      {id: 8, name: 'Steve',  phone: '010-6677-8765'},  
    ],
    102: [
      {id: 9, name: 'Bella',  phone: '010-9876-5432'},
      {id: 10, name: 'Max',   phone: '010-5555-9999'},
      {id: 11, name: 'Ivy',   phone: '010-3333-2222'},
      {id: 12, name: 'Luke',  phone: '010-4444-6666'},  
    ]
  }
};
```

## Auto Increment 
Auto Increment allows a unique number to be generated automatically when a new record is inserted into a table. Simple! but it has problems...
- 값의 고갈: DBMS가 지원하는 Integer 최댓값을 초과할 수 없습니다.
- 경합 조건에서의 성능 문제: 동시에 많은 Row가 생성이 될 때 다음 값을 기다려야 하므로 성능 저하가 발생할 수 있다.
- 분산 시스템에서의 떨어지는 확장성: 데이터베이스를 여러 개 사용할 경우 정수 키값이 여러 노드에서 중복으로 생성될 수 있다.
- 데이터 보안: 외부 공격자가 다른 사용자의 ID를 쉽게 예측할 수 있습니다. 숫자만 바꾸면 되니까요
- 삭제: 레코드가 삭제되었을 때, Auto Increment 필드는 삭제된 레코드의 값을 재사용하지 않습니다. 이로 인해 값의 사용이 비효율적일 수 있습니다.

## UUID

## Timeflake