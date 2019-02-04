layout: true
.top-line[]

---
class: center, middle
# Room (데이터베이스)

---
## Contents
- Room 개요


---
## Room 개요
- Room
    - SQLite를 쉽게 사용할 수 있는 데이터베이스 객체 매핑 라이브러리SQL
    - 쉽게 Query를 사용할 수 있는 API를 제공
    - Query를 컴파일 시간에 검증함
    - Query결과를 LiveData로하여 데이터베이스가 변경될 때 마다 쉽게 UI를 변경할 수 있음
- Room의 주요 3요소
    - **@Database**: 클래스를 데이터베이스로 지정하는 annotation, RoomDatabase를 상속 받은 클래스여야 함
        - Room.databaseBuilder를 이용하여 인스턴스를 생성함
    - **@Entity**: 클래스를 테이블 스키마로 지정하는 annotation
    - **@Dao**: 클래스를 DAO(Data Access Object)로 지정하는 annotation
        - 기본적인 insert, delete, update SQL은 자동으로 만들어주며, 복잡한 SQL은 직접 만들 수 있음

---
## gradle 파일 설정
- Room은 안드로이드 아키텍처에 포함되어 있음
- 사용하기위해 build.gradle 파일의 dependencies에 아래 내용을 추가해야 함
    - Androidx 사용하는 경우를 가정함, Android Studio와 SDK는 최신 버전으로 사용
    - Androidx를 사용하지 않는다면 메뉴 Refactor > Migrate to AndroidX 를 이용하여 사용할 수 있음

```bash
    // Room components
    implementation "androidx.room:room-runtime:2.0.0-beta01"
    annotationProcessor "androidx.room:room-compiler:2.0.0-beta01"
    androidTestImplementation "androidx.room:room-testing:2.0.0-beta01n"
    
    // Lifecycle components
    implementation "androidx.lifecycle:lifecycle-extensions:2.0.0-beta01"
    annotationProcessor "androidx.lifecycle:lifecycle-compiler:2.0.0-beta01"
```

.footnote[https://github.com/jyheo/android-lecture-examples/blob/master/room/build.gradle]

---
## Entity 생성
- Entity는 테이블 스키마 정의
- CREATE TABLE student_table (id INTEGER PRIMARY KEY, name TEXT NOT NULL); 

```java
@Entity(tableName = "student_table")    // 테이블 이름을 student_table로 지정함
public class Student {                  // (지정하지 않으면 클래스 이름 사용)

    @PrimaryKey(autoGenerate = true)    // Primary Key를 지정하는 annotation
    int id;

    @NonNull                            // 이 속성은 null이 될 수 없음
    @ColumnInfo(name = "name")          // 테이블 속성 이름은 name으로 지정함
    private String mName;

    Student(@NonNull String name) {mName = name;}  // 생성자

    public String getName() {return mName;}  // getter 메소드
}
```

.footnote[https://github.com/jyheo/android-lecture-examples/blob/master/room/src/main/java/com/example/roomexample/Student.java]

---
## DAO 생성
- DAO는 interface나 abstract class로 정의되어야 함
- Annotation에 SQL 쿼리를 정의하고 그 쿼리를 위한 메소드를 선언
- 가능한 annotation으로 @Insert, @Update, @Delete, @Query가 있음
    - ```@Query("SELECT * from table") List<Data> getAllData();```
- @Insert, @Update, @Delete는 SQL 쿼리를 작성하지 않아도 컴파일러가 자동으로 생성함    
    - @Insert나 @Update는 key가 중복되는 경우 처리를 위해 onConflict를 지정할 수 있음
        - OnConflictStrategy.ABORT: key 충돌시 종류
        - OnConflictStrategy.IGNORE: key 충돌 무시
        - OnConflictStrategy.REPLACE: key 충돌시 새로운 데이터로 변경
    - @Update나 @Delete는 primary key에 해당되는 튜플을 찾아서 변경/삭제 함
    
---
## DAO 생성
- @Query로 리턴되는 데이터의 타입을 LiveData<>로 하면, 나중에 이 데이터가 업데이트될 때 Observer를 통해 할 수 있음
    - ```@Query("SELECT \* from table") LiveData<List<Data>> getAllData(); ```
- @Query에 SQL을 정의할 때 메소드의 인자를 사용할 수 있음
    - ```@Query("SELECT \* FROM student_table WHERE name = :sname") 
    LiveData<List<Student>> selectStduentByName(String sname);```
    - 인자 sname을 SQL에서 :sname으로 사용
---
## DAO 생성


```java
@Dao
public interface MyDao {

    @Insert(onConflict = OnConflictStrategy.REPLACE)  // INSERT, key 충돌이 나면 새 데이터로 교체
    void insertStudent(Student student);  

    @Query("DELETE FROM student_table")
    void deleteAllStudent();

    @Query("SELECT * FROM student_table")
    LiveData<List<Student>> getAllStudents();        // LiveData<> 사용

    @Query("SELECT * FROM student_table WHERE name = :sname")   // 메소드 인자를 SQL문에서 사용할 때 :을 씀
    LiveData<List<Student>> selectStduentByName(String sname);

    @Delete
    void deleteStudent(Student... student); // primary key is used to find the student

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    void insertTeacher(Teacher teacher);

    //@Query("SELECT ")
}
```