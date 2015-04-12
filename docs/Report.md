# ***Game Ai Là Triệu Phú***

####1. Yêu cầu bài toán: 

* Mỗi câu hỏi có 4 đáp án và chỉ có 1 đáp án đúng. Giới hạn độ dài câu hỏi là 150 ký tự, độ dài đáp án là 50 ký tự.
* Có vấn đề mới phát sinh là có phân loại câu hỏi theo 3 mức: ** Dễ, Trung bình,Khó **.
* Lưu được thông tin cấu hình cho game: Số tiền đạt được ứng với số câu hỏi trả lời được (Ví dụ: trả lời được 1 câu được 500k, 2 câu được 800k, …).
* Giả lập trợ giúp người chơi (50-50, trợ giúp từ khán giả, gọi điện cho người thân).
* Lưu thông tin người chơi bao gồm: Họ tên, Thời điểm chơi, Số tiền đạt được.
* Quản lý game cần thực hiện các chức năng quản lý
####2. Ý tưởng:
* Tạo 2 bảng cơ sở dữ liệu. Bảng 1 là bảng `questions` gồm các thuộc tính như sau: **idQuestion**, **question**, **ask1**, **ask2**,**ask3**, **ask4**,**result**, **idLevel**  Và bảng 2 là bảng `levels` gồm có.**namelevel**,**idlevel**
* Mối quan hệ giữa 2 bảng là : `questions` và `levels` là mối quan hệ 1 nhiều thông qua khóa ngoại **idLevel**
####3. Các câu lệnh tạo bảng:


##### Tạo bảng Questions(Theo yêu cầu bài toán thì levels được thêm sau)
```
create table Questions(
idQuestion serial not null,
question character varying(150) not null,
ask1 character varying(50) not null,
ask2 character varying(50) not null,
ask3 character varying(50) not null,
ask4 character varying(50) not null,
state character varying(50) not null,
--idLevel integer not null,
constraint questions_pkey primary key (idQuestion),
--constraint question_level_fkey foreign key (idLevel)
--references Levels(idLevel)
);
```
**Database questions**
![image](images/dbq.PNG "db")
---------------------------------------
##### Tạo bảo levels
```
create table Levels(
idLevel serial not null,
levelName character varying(10) not null,
constraint idLevel_pkey primary key (idLevel)
);
```

**Database level**
![image](images/level.PNG "lv")
####4. Các câu lệnh truy vấn giải quyết bài toán:
* **1.1 Câu lệnh insert 100 câu hỏi(chưa có level) viết hàm xử lý**
```
CREATE OR REPLACE
FUNCTION auto_add_question(quantity integer)
RETURNs void AS
$BODY$
declare 
	i int;  x int; y int;
BEGIN for i IN 1..$1 LOOP
	x = round(random()*100);
	y = round(random()*100);
INSERT INTO questions(question,ask1,ask2,ask3,ask4,result) values
	(CONCAT(x,'+',y,'=?'),x+y,x-y,x-y+1,x+y+1,x+y);
END LOOP;
END;
$BODY$
LANGUAGE plpgsql;


select auto_add_question_math(100);

```
* **1.2 Truy vấn 1 câu hỏi bất kỳ và lấy ra được nội dung câu hỏi, 4 đáp án, đáp án đúng:**

```
select q.question, q.ask1, q.ask2,q.ask3,q.ask4, q.result from questions q offset random()*100 limit 1;
```
**Database random **
![image](images/random1q.PNG "lv")

* ** 1.3Truy vấn 1 câu hỏi bất kỳ và lấy ra được nội dung câu hỏi, 4 đáp án,đáp án đúng với dữ liệu với nội dữ liệu không trùng nhau (Sử dụng JSON):**

```
select row_to_json(row) from (select * from questions order by random()*100 limit 1) row;
```
**Database json **
![image](images/json.PNG "lv")

* **2.1 Hãy cập nhật thông tin bảng với yêu cầu mới thêm (Các bảng trước đó đã có và có dữ liệu):**

```
ALTER TABLE questions ADD COLUMN idLevel integer,
ADD CONSTRAINT question_levels_fkey FOREIGN KEY (idLevel) REFERENCES Levels(idLevel);
```
* **2.2 Truy vấn số lượng câu hỏi theo mỗi mức độ khó để kiểm tra số lượng câu hỏi cho mỗi mức độ khó đã bằng nhau chưa:**

```
select l.levelname, count(q.idLevel) as dem
from levels as l  right join
questions as q on l.idLevel=q.idLevel
group by l.levelname;
```
**Database Show count level **
![image](images/demlevel.PNG "lv")
* **2.3 Truy vấn lấy ngẫu nhiên được 1 câu hỏi thuộc mức độ Dễ (Trung bình, Khó):**

```
select q.question,l.levelname 
from questions q left join levels as l on l.idLevel=q.idLevel 
offset random()*100 limit 1;
```
**Database random question level **
![image](images/levelrandom.PNG "lv")

* **2.4 Truy vấn lấy ra ngẫu nhiên 15 câu hỏi và sắp xếp theo thứ tự độ khó tăng dần (mỗi độ khó có 5 câu hỏi sử dụng UNION):**

```
(select question,idlevel from questions where idlevel = 1  ORDER BY random() LIMIT 5)
UNION ALL
(select question,idlevel from questions where idlevel = 2  ORDER BY random() LIMIT 5 )
UNION ALL
(select question ,idlevel from questions where idlevel = 3  ORDER BY random() LIMIT 5);
```

* **3.1 Nhập dữ liệu cấu hình cho game (15 mốc câu hỏi như Ai là triệu phú) (tạo bảng điểm)**

```
create table goals(
 idGoal serial Not Null,
  money character varying(100) NOT NULL,
  CONSTRAINT topic_pkey PRIMARY KEY(idGoal)
);

--insert du lieu
Insert into goals(money) values('500000');
Insert into goals(money) values('800000');
Insert into goals(money) values('1000000');
....

```
* **4.1 Truy vấn lấy được ngẫu nhiên 2 trong số 4 đáp án của 1 câu hỏi bất kỳ, trong đó bắt buộc phải chứa đáp án đúng:):**

```
CREATE OR REPLACE FUNCTION bo_di_2_phuong_an_sai(id INTEGER)
   RETURNS TABLE(
	name CHARACTER VARYING(150),
	t CHARACTER VARYING(50),
	f CHARACTER VARYING(50)
   ) AS $$    
DECLARE 
     rand INTEGER;
BEGIN
    rand = round(random() * 10);
     rand = rand % 3;
    CASE 
      WHEN rand = 0 THEN RETURN QUERY SELECT questions.question,ask1,ask2 FROM questions WHERE questions.idquestion = $1;
      WHEN rand = 1 THEN RETURN QUERY SELECT questions.question,ask1,ask3 FROM questions WHERE questions.idquestion = $1;
      ELSE RETURN QUERY SELECT questions.question,ask1,ask4 FROM questions WHERE questions.idquestion = $1;
    END CASE;
END 
  $$
LANGUAGE plpgsql;
 
SELECT bo_di_2_phuong_an_sai(100);
```

* **4.2 Truy vấn ngẫu nhiên tỉ lệ khán giả chọn đáp án của 1 câu hỏi theo % (Ví dụ: Đáp án A – 20%, B – 30%, C – 40%, D – 10%): **

```
--ham tao randoms
 CREATE OR REPLACE FUNCTION randoms(numeric, numeric)
   RETURNS numeric AS
   $$
    SELECT ($1 + ($2 - $1) * random())::numeric;
 $$ LANGUAGE 'sql' VOLATILE;


--4.2 Hoi y kien khan gia
CREATE OR REPLACE FUNCTION hoi_y_kien_khan_gia()
   RETURNS TABLE(
       A INTEGER,
       B INTEGER,
       C INTEGER,
       D INTEGER
   ) AS $$
 DECLARE 
     d INTEGER;
     a INTEGER;
     b INTEGER;
     c INTEGER;
 BEGIN
   d = 100;
   a = round(randoms(0,d));
   d = d - a;
   b = round(randoms(0,d));
   d = d - b;
   c = round(randoms(0,d));
   d = d - c;
   RETURN QUERY
     SELECT A,B,C,D;
 END
 $$
   LANGUAGE plpgsql;
 SELECT hoi_y_kien_khan_gia();

```
* **4.3 Truy vấn ngẫu nhiên đáp án cho 1 câu hỏi: 5đ (Câu hỏi gọi trợ giúp người thân) **

```
```

* **5.1 Nhập dữ liệu mẫu 100 người chơi:**

```
create table users(
 iduser serial Not Null,
  nameUser character varying(100) NOT NULL,
  startDate time without time zone NOT NULL,
  totalGoal character varying(100) NOT NULL,
  CONSTRAINT user_pkey PRIMARY KEY(iduser)

)
Insert into users(nameUser,startDate,totalGoal) values
('Vũ Công Hiếu',current_timestamp,'1200000');
Insert into users(nameUser,startDate,totalGoal) values
('Nguyễn Đăng Khiêm',current_timestamp,'1200000');
```
* **5.2 Truy vấn lấy ra 10 người chơi đạt điểm cao nhất, sắp xếp theo thứ tựđiểm cao giảm dần (nếu 2 người chơi có cùng điểm số thì người chơi sau sẽ được xếp ở vị trí cao hơn):**

```
select * from goals
order by money
Limit 10;
```
* **6.1 Tìm kiếm câu hỏi theo id, theo từ khóa trong câu hỏi, theo từ khóa trong câu trả lời:**



