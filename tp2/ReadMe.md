# TP2

| Timing | Session N° 1 (User1)   | Session N° 2 (User2) |Résultat | 
| :----: | :----: |:----:|:----:|
| t0 | ``` SELECT ENAME, SAL FROM EMP WHERE ENAME IN ('Mohamed','Hichem');``` |||
| t1 | ``` UPDATE EMP SET SAL = 4000 WHERE ENAME ='Hichem'; ``` |------|Updated|
| t2 | ------ |```UPDATE EMP SET SAL = SAL + 1000 WHERE ENAME ='Mohamed';```|Updated|
| t3 | ```UPDATE EMP SET SAL = SAL + 1000 WHERE ENAME ='Mohamed';```|Blocked, User 1 updated this row without finalizing the update with "commit"|
| t4 | ------ |```UPDATE EMP SET SAL = SAL + 1000 WHERE ENAME ='Hichem';```|La session 1 va detecter l'interblocage|
| t5 | ```Commit;``` |------|Session 2: --> 1 row updated.|
| t6  |```UPDATE EMP SET SAL = SAL + 1000 WHERE ENAME ='Mohamed';```| ------|User1 blocked, User 2 updated this row without "commit"|
| t7 |  ------ |```Commit;```| Updated in Session 1|
| t8 | ------ |```SELECT ENAME, SAL FROM EMP WHERE ENAME IN ('Mohamed','Hichem', 'Maaoui');```|displays all the committed changes|


### Demo Niveau d'isolation  READ COMMITTED 

| Timing | Session N° 1  | Session N° 2 |Résultat | 
| :----: | :----: |:----:|:----:|
| t0| ``` SELECT ENAME, SAL FROM EMP WHERE ENAME IN ('Mohamed','Hichem');``` |||
| t1 | ``` UPDATE EMP SET SAL = 4000 WHERE ENAME ='Hichem'; ``` |------|Updated|
| t2 | ------ |```SET TRANSACTION ISOLATION LEVEL READ COMMITTED;```|Transaction set|
| t3 | ------ |```SELECT ENAME, SAL FROM EMP WHERE ENAME IN ('Mohamed','Hichem');```|la requete de User 1 n'est pas effectuée, pas de "commit" alors "verrouillé"|
| t4 | ------ |```UPDATE EMP SET SAL = 3800 WHERE ENAME ='Mohamed';```|Updated|
| t5 | ```Insert into EMP (EMPNO,ENAME,JOB,MGR,HIREDATE,COMM,DEPTNO) values ('9999','Maaoui','Magician',null,to_date('17/02/2021','DD/MM/RR'),null,'10');``` |------|Row created|
| t6 | ------ |```SELECT ENAME, SAL FROM EMP WHERE ENAME IN ('Mohamed','Hichem', 'Maaoui');```|inserted row isn't shown|
| t7 | ------ |```UPDATE EMP SET SAL = 5000 WHERE ENAME ='Hichem';```|blocked|
| t8 | ```Commit;``` |------|la requete precedente de User 2 est effectuée|
| t9 | ------ |```SELECT ENAME, SAL FROM EMP WHERE ENAME IN ('Mohamed','Hichem', 'Maaoui');```|displays the previous update + the previously inserted row by User1|
| t10| ------ |```COMMIT;```|commit complete|
| t11| ```SELECT ENAME, SAL FROM EMP WHERE ENAME IN ('Mohamed','Hichem', 'Maaoui');```|------|changes by User2 are now also visible to User1|


### Demo Niveau d'isolation SERIALIZABLE ;

| Timing | Session N° 1  | Session N° 2 |Résultat | 
| :----: | :----: |:----:|:----:|
| t0| ``` SELECT ENAME, SAL FROM EMP WHERE ENAME IN ('Mohamed','Hichem');``` |||
| t1| ``` UPDATE EMP SET SAL = 4000 WHERE ENAME ='Hichem'; ``` |------|Updated|
| t2| ------ |```SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;```|Transaction set|
| t3| ------ |```SELECT ENAME, SAL FROM EMP WHERE ENAME IN ('Mohamed','Hichem');```|la requete de User 1 n'est pas effectuée, pas de "commit" alors "verrouillé"|
| t4| ------ |```UPDATE EMP SET SAL = 3800 WHERE ENAME ='Mohamed';```|Updated|
| t5| ```Insert into EMP (EMPNO,ENAME,JOB,MGR,HIREDATE,COMM,DEPTNO) values ('9999','Maaoui','Magician',null,to_date('17/02/2021','DD/MM/RR'),null,'10');``` |------|Row created|
| t6| ```COMMIT;```|------ |commit complete|
| t7|```SELECT ENAME, SAL FROM EMP WHERE ENAME IN ('Mohamed','Hichem', 'Maaoui');```| ------ |displays the changes made by User 1|
| t8| ------ |```SELECT ENAME, SAL FROM EMP WHERE ENAME IN ('Mohamed','Hichem', 'Maaoui');```|displays the changes made *only* by User 2 despite the previous commit from User 1|
| t9| ------|```Commit;``` |commit complete|
| t10|```SELECT ENAME, SAL FROM EMP WHERE ENAME IN ('Mohamed','Hichem', 'Maaoui');```| ------ |displays the changes made by both users|
| t11| ------ |```SELECT ENAME, SAL FROM EMP WHERE ENAME IN ('Mohamed','Hichem', 'Maaoui');```|displays the changes made by both users|
| t12| ------ | ```COMMIT;```|------|
| t13| ``` UPDATE EMP SET SAL = 5000 WHERE ENAME ='Maaoui'; ``` |------|Updated|
| t14| ------ |```SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;```|Transaction set|
| t15| ------ |```UPDATE EMP SET SAL = 5200 WHERE ENAME ='Maaoui';```|Blocked|
| t16| ```COMMIT;``` |------|commit complete, can't serialize access for previous transaction|
| t17| ------ |```ROLLBACK;```|rollback complete|
| t18| ------ |```SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;```|Transaction set|
| t19| ------ |```SELECT ENAME, SAL FROM EMP WHERE ENAME IN ('Mohamed','Hichem', 'Maaoui');```|Update by User 1 is displayed|
| t20| ``` UPDATE EMP SET SAL = 5200 WHERE ENAME ='Maaoui'; ``` |------|Updated|
| t21| ```COMMIT;``` |------|commit complete|
| t22| ------ | ```COMMIT;```|commit complete|
| t23| ------ |```SELECT ENAME, SAL FROM EMP WHERE ENAME IN ('Mohamed','Hichem', 'Maaoui');```|previous update by User 1 is displayed|
