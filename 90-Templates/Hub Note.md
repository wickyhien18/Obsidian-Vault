
## Dự án liên quan
```dataview
TABLE status as "Trạng thái", type as "Loại"
FROM "20-Projects"
WHERE hub = "web-node"
SORT file.name ASC
```

## Kiến thức liên quan
```dataview
TABLE status as "Trạng thái"
FROM "10-Concepts"
WHERE hub = "web-node"
SORT file.name ASC
```