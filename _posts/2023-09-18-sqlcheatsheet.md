---
layout: post
title:  "sql cheat sheet"
date:   2023-09-18 19:26:01 +0900
tag: c++
---

기본 문법도 못 외워서 맨날 검색하는 나를 위해

# select 

```sql
SELECT DEPTNAME, MGRNO        FROM CORPDATA.DEPARTMENT        WHERE DEPTNO = 'C01'
```



# insert 

```sql
INSERT INTO table-name (column1, column2, ... ) VALUES (value-for-column1, value-for-column2, ... )

INSERT INTO CORPDATA.EMPTIME (EMPNUMBER, PROJNUMBER, STARTDATE, ENDDATE) SELECT EMPNO, PROJNO, EMSTDATE, EMENDATE FROM CORPDATA.EMPPROJACT
```


# update

```sql
UPDATE table-name SET column-1 = value-1, column-2 = value-2, ...  WHERE search-condition ...
```

# delete

```sql
DELETE FROM CORPDATA.EMPLOYEE WHERE WORKDEPT = 'D11'
```



