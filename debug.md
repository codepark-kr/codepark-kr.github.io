---
layout: page
title: /debug&refactor
permalink: /debug-refactor
---

### Debug & Refactor (frequent update)

`2021.06.01-2021.11.21`
@yb park 

---

CONTENTS
[3. Debug](#3-debug)
    [3-1. [cat.NCDC OPEN API] func. ApiInfo012](#3-1-catncdc-ope-api-func-ApiInfo012)
    [3-2. [cat.Eclipse] java.lang.OutOfMemoryError: PermGen space](#3-2-cateclipse-javalangoutofmemoryerror-permgen-space)
    [3-3. [Tibero5/Oracle DBMS] Alter System Kill Session](#3-3-tibero5oracle-dbms-alter-system-kill-session)
    [3-4. [Tibero5/Oracle DBMS] Recover & Restore for updated wrong data](#3-4-tibero5oracle-dbms-recover--restore-for-updated-wrong-data)
[4. Methodology of Debugging with Eclipse](#4-methodology-of-debugging-with-eclipse)
    [4-0. Reference](#4-0-reference)
    [4-1. Introduction](#4-1-introduction)
    [4-2. Explanation](#4-2-explanation)

---

### 3-1. [cat.NCDC OPE API] func. ApiInfo012
***Problem***
1. `java.io.FileNotFoundException` occured when request to get response data from API
2. got HTML instead of JSON when request to getting data from API
3. The response from API contained `NO-DATA` code, also unincluded the requested data 


***Cause***
1. The IP address was hard-coded in `OPENAPI.END_POINT_PATH.HTTP.TEST@Config.properties`, also it was incorrect.
2. The local-file path was hard-coded in  `main@ApiInfo012Controller` also it wasn't modified.
3. The header or request parameter for data type of result was missing.
4. The original base code contained wrong if-conditional statements. so It caused data missing.

***Solutions***
1-1. modify the request-URL via `@config.properties` 

{% highlight bash %}
# SYNTAX : http://][ip address for request][PORT][group ID] 
OPENAPI.END_POINT_PATH.HTTP.TEST=http://localhost:8000/KMA_OPENAPI_2019/
{% endhighlight %}

1-2. modify the path for Cacher-path-statements via `main@ApiInfo012Controller.java`

{% highlight java %}
// SYNTAX :  DFS[cat.abbr]Cacher.init(frontpach + [correct path], false);
DFSOdamCacher.init(frotpach + "C:/Users/codep/Downloads/DFS/GEMD/VSRT", false);
DFSVsrtCacher.init(frontpach + "C:/Users/codep/Downloads/DFS/GEMD/VSRT", false);
ㅡ
ㅡ + "C:/Users/codep/Downloads/DFS/GEMD/VSRT", false);
{% endhighlight %}

1-3. add header `Accept: application/json` or `dataType=JSON` to request parameter

{% highlight java %}
// to adding request params, just add [&dataType=JSON] to request uri. 
setHeader("Accept", "application/json");
{% endhighlight %}

1-4. annotate the wrong if statement via `initDFSVsrtWeather@DFSPointVsrtData.java`

{% highlight java %}
// the weather-ArrayList generated and no values allocated. 
// but the if-conditional statement checking that arrayLists size.
// so it must be pass the conditional statement, cannot get any values.
public void initDFSVsrtWeather() {
		int fctCnt = Config_Vsrt.FCT_CNT[Config_Vsrt.V_SKY][this.getAnnTime().getTmSeq()];
		weather = new ArrayList<ApiInfo012Vo>();
		Calendar kstCal = (Calendar)dfsTs.getKstCal().clone();
		kstCal.set(Calendar.MINUTE, 30);
		String kstBaseTm = StringUtils.getFormattedDate(kstCal.getTime(), "yyyyMMddHHmm");
		kstCal.set(Calendar.MINUTE, 0);
//		if(weather.size() > 0) { <- problem solve
{% endhighlight %}

---

### 3-2. [cat.Eclipse] java.lang.OutOfMemoryError: PermGen space

***Reference***
[FAQ How do I increase the permgen size available to Eclipse?](https://wiki.eclipse.org/FAQ_How_do_I_increase_the_permgen_size_available_to_Eclipse%3F)

***Problem***
When trying to run the local-server with eclipse, the Permgen Exception occurred too often so the work-efficiency getting lower. (when this exception occurred then needed to project clean and restart - sometimes had to shut down eclipse IDE and restart.)   

***Cause***
Out of memory. 
so allocated memory to Eclipse is not enough for running two servers both at same time.

***Solution***
—   extracted
If you see `java.lang.OutOfMemoryError: PermGen space` errors, you need to increase the permanent generation space available to Eclipse. Permgen is the permanent generation of objects in the VM (Class names, internalized strings, objects that will never get garbage-collected). An easy, if somewhat memory-hunger fix is to enlarge the maximum space for these object by adding  :

{% highlight bash %}
-XX:MaxPermSize=128M
{% endhighlight %}

as an argument to the JVM when starting Eclipse, This recommended way to do this is via your `eclipse.ini` file. Alternatively, you can invoke the Eclipse executable with command-line arguments directly, as in 

{% highlight bash %}
eclipse [normal arguments] -vmargs -XX:permSize=64M -XX:MaxPermSize=128M
{% endhighlight %}

Note 1: The arguments after `-vmargs` are directly passed to the VM. Run `java -X` for the list options your VM accepts. Options starting with `-X` are implementation-specific and may not be applicable to all JVMs. (although they do work with the Sun/Oracle JVMs.)  

*Note 2: Also Oracle java 8 does not have a separate permanent generation space any more. The `-XX:`(Max)Permsize option makes no difference (the JVM will ignore it, so it can still be present.)*

---

### 3-3. [Tibero5/Oracle DBMS] Alter System Kill Session

***Problems***
Infinite Loading occurred when trying execute DML statement with Tibero5 DBMS.

***Cause***
The SQL session still didn't closed, 
i.e. commit or rollback no executed after they request DML statement at before.

***Solutions***
Kill the System Session and try again.

{% highlight sql %}
// look up the SID(Session ID) and serial#
SELECT * FROM V$SESSION; 

// kill the system session
// syntax : ALTER SYSTEM KILL SESSION '[SID], [serial#]';
ALTER SYSTEM KILL SESSION '74,192135';
{% endhighlight %}

***Reference***
[사용자 세션의 강제 종료](http://www.gurubee.net/lecture/1156)

---

### 3-4. [Tibero5/Oracle DBMS] Recover & Restore for updated wrong data

***Problems***
**The new data for only one row replaced to data of all rows!**

***Cause***
Unexpected semicolon placed between `SET [COL] = [DATA]` and `WHERE [COL] = [CONDITION]`!

***Solutions***
1. Increase `UNDO_RETENTION` . It makes the retention time longer. *(ATCH.1)*
2. Get retained data by `TIMESTAMP`. *(ATCH.2)*
3. Restore replaced data by retained data, with conditional statement. *(ATCH.3)* 
4. It works. Thanks god.

{% highlight sql %}
-- (ATCH.1)
ALTER SYSTEM SET UNDO_RETENTION = [SECONDS];
{% endhighlight %}

{% highlight sql %}
-- (ATCH.2)
SELECT * FROM [TABLE_NAME] AS OF TIMESTAMP (SYSTIMESTAMP - INTERVAL '[NUM]'MINUTE);
-- e.g.
SELECT * FROM TB_OPERATION_INFO AS OF TIMESTAMP(SYSTIMESTAMP - INTERVAL '120'MINUTE) ORDER BY OPERATION_CD ASC;
{% endhighlight %} 

{% highlight sql %}
-- (ATCH.3)
UPDATE [TABLE_NAME] [ALIAS] SET
	[COLUMN NAME FOR WANNA RECOVER] = (
		SELECT [COLUMN NAME FOR WANNA RECOVER]
		FROM [TABLE NAME] AS OF TIMESTAMP(SYSTIMESTAMP - INTERVAL '[NUM]'MINUTE)
		WHERE [COLUMN NAME WANNA RECOVER] = [ALIAS].[COLUMN NAME WANNA RECOVER]
	);
{% endhighlight %}

***Reference***
[ORACLE DATA 오라클 데이터 복구 (일정 시간 이내)](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=skyupup&logNo=220660235835)
[오라클 DELETE, UPDATE 후 COMMIT 한 데이터 복구하는 방법](https://unabated.tistory.com/entry/%EC%98%A4%EB%9D%BC%ED%81%B4-DELETE-UPDATE-%ED%9B%84-COMMIT-%ED%95%9C-%EB%8D%B0%EC%9D%B4%ED%84%B0-%EB%B3%B5%EA%B5%AC%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95)
[[oracle] TIMESTAMP 데이터 복원, 복구](https://dlevelb.tistory.com/704)

---

## 4. Methodology of Debugging with Eclipse
### 4-0. Reference
[Debugging the Eclipse IDE for Java Developers | The Eclipse Foundation](https://www.eclipse.org/community/eclipse_newsletter/2017/june/article1.php)

### 4-1. Introduction
***Debugging*** is the routine process of locating and removing bugs, errors or abnormalities from programs. It's a must have skill for any Java Developer because it helps to find subtle bug that are not visible during code reviews or that only happens when a specific condition occurs. The Eclipse Java IDE provides many debugging tools and views grouped in the Debug Perspective to help the you as a developer debug effectively and efficiently.

### 4-2. Explanation
1. **Launching and Debugging a Java program**
A Java program can be debugged simply by right clicking on the Java editor class file from Package explorer. Select Debug As → Java Application , or use the shortcut `Alt + Shift +D, J` instead.

![img](/uploads/debug%20(7).png)
![img](/uploads/debug%20(6).png)
Either actions mentioned above creates a new ***Debug Launch Configuration*** and uses it to start the Java Application.

![img](/uploads/debug%20(5).png)
In most cases, users can edit and save the cod while debugging without restarting the program. This works with the supports of **HCR**(Hot Code Replacement), which has been specifically added as a standard Java Technique to facilitate experimental development and to foster iterative trial-and-error coding.

1. **Breakpoints** 
A breakpoint is a signal that tells the debugger to temporarily suspend execution of your program at a certain point in the code. To define a breakpoint in your source code, right-click in the left margin in the Java editor and select ***Toggle Breakpoint.*** Alternatively, you can double-click on this position. 

![img](/uploads/debug%20(4).png)
The Breakpoints view allows you to delete and deactivate Breakpoints and modify their properties.
![img](/uploads/debug%20(3).png)

All breakpoints can be enable/disable using ***Skip App Breakpoints.*** Breakpoints can also be imported/exported to and from a workspace.

**3. Debug Perspective**
The debug perspective offers additional views that can be used to troubleshoot and application like Breakpoints, Variables, Debug, Console etc. When a Java program is started Debug mode, users are prompted to switch to the debug perspective. 

![img](/uploads/debug%20(2).png)

- **Debug View** - Visualizes call stack and provides operations on that.
- **Breakpoints View** - Show all the breakpoints.
- **Variables/Expressions View** - Shows the declared variables and their values. Press `Ctrl + Shift + d` or `Ctrl + Shift + i` on a selected variables or expression to show it value. You can also add a permanent watch on an expression/variables that will then be shown in the *Expressions View* when debugging is on.
- **Display view** - Allows to Inspect the value of variable, expression or selected text during debugging.
- **Console View** - Program output is shown here.

**4. Stepping Commands**

The Eclipse Platform helps developers debug by providing buttons in the toolbar and key binding shortcuts to control program execution. For more information about debugging then see also : [Eclipse Stepping Commands Help](https://help.eclipse.org/2021-03/index.jsp?topic=%2Forg.eclipse.jdt.doc.user%2Ftasks%2Ftask-stepping.htm). *Also when trying to debugging  with stepping commands, try to follow the data-flow from bottom to top.* 

![img](/uploads/debug%20(1).png)
---