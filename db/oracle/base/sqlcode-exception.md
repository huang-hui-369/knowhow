# sqlcode

sqlcode是oracle返回的sql语句的执行的返回码，
如果有异常的话，返回系统预先定义的异常代码

oracle给用于预留了-20000 -- -20999的用户定义的异常代码

如果要想通过sqlCODe捕获用户定义的异常代码
要么在定义异常时，通过初始化设置异常代码
```sql
myexception EXCEPTION;
pragma exception_init(myexception, -20002);
```
要么通过函数 RAISE_APPLICATION_ERROR(-20001,'my exception');
来抛出异常。


```sql
-- sql developer DBMS_OUTPUT.PUT_LINE 出力のため
SET SERVEROUTPUT ON;

DECLARE
          v_deptno NUMBER := 500;
          v_name VARCHAR2 (20) := 'Testing';
          myexception EXCEPTION;
          pragma exception_init(myexception, -20002);
         BEGIN
            -- RAISE_APPLICATION_ERROR(-20001,'my exception');
            RAISE myexception;
         EXCEPTION
           WHEN myexception THEN
             DBMS_OUTPUT.PUT_LINE ('catch myexception');
             DBMS_OUTPUT.PUT_LINE (SQLERRM);
             DBMS_OUTPUT.PUT_LINE (SQLCODE);
           WHEN OTHERS THEN
             DBMS_OUTPUT.PUT_LINE ('catch RAISE_APPLICATION_ERROR my exception');
             DBMS_OUTPUT.PUT_LINE (SQLERRM);
             DBMS_OUTPUT.PUT_LINE (SQLCODE);
        END;
        
        /
```