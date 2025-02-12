<img src="https://companieslogo.com/img/orig/MDB_BIG-ad812c6c.png?t=1648915248" width="50%" title="Github_Logo"/> <br>


# MongoDB Atlas Training Data Modeling & Advanced MQL

### MQL

#### Reviewing Query
Mongosh을 이용하여 배포된 클러스터에 접속 후 다음을 실행 하여 줍니다.

Aggregation을 진행 하기 전에 MQL의 find를 이용하여 데이터를 filter하고 데이터를 정렬 할 수 있습니다.  

##### Basic MQL 작성

filter
projection
sort

find operation에서 원하는 데이터만을 반환 받기 위한 방법으로 기본적인 내용을 리뷰 합니다.

sample_mflix 데이터베이스의 movies 컬렉션에서 1933년에 개봉한 영화 중 가장 많은 상을 수상한 영화를 반환하는 쿼리를 작성 하고, title과 award 내용을 출력 합니다.   

결과는 다음과 같습니다.  
```` 
[
  {
    title: 'State Fair',
    awards: { text: 'Nominated for 2 Oscars. Another 2 wins.' }
  }
]
```` 

Query에 .next().awards.text 를 이용하여 award에 대한 내용만을 출력 할 수 있습니다.   

mongosh에서 한 줄에 Query를 전부 작성하는 것보다 매개 변수를 이용하여 처리하고 실행 하는 것이 좋은 방법 입니다. 

```` 
db = db.getSiblingDB("sample_mflix")

query = {year:1933}
projection = {_id:0,"title":1, "awards.wins":1}
sortOrder = {"awards.wins":-1}

doc = db.movies.find(query,projection).sort(sortOrder).limit(1).toArray()

```` 


##### $elemMatch를 이용

MQL의 구조는 단순하여 사용이 어렵지 않습니다. 이 중 유용한 기능은 $elemMatch 입니다.
$elemMatch 연산자는 배열에 대해 쿼리하는데 사용되며 scope를 연산하는데에 사용 됩니다. 

다음과 같이 간단한 이름을 정보를 저장한 데이터를 작성 합니다. 

```` 
r = {
  people : [
          { firstname: "john", lastname: "wayne" },
          { firstname: "jayne" , lastname: "doe" }
   ]
}
db.a.drop()
db.a.insertOne(r)

```` 

"John Doe"라는 이름을 검색 하는 Query를 작성 하면 다음과 같이 작성 할 수 있습니다.

```` 
db.a.find({"people.firstname":"john","people.lastname":"doe"})
```` 

결과는 다음과 같습니다.

```` 
[
  {
    _id: ObjectId("67abfdebd4d7d2620cdcac5b"),
    people: [
      { firstname: 'john', lastname: 'wayne' },
      { firstname: 'jayne', lastname: 'doe' }
    ]
  }
]
```` 

배열내의 엘리먼트에서 검색하는 경우 firstname에 "john"이 있는지 검색하고 lastname에 "doe"가 있는 지 검색 하게 되어 원하는 검색 결과가 출력 되지 않습니다. 
정확한 이름으로 검색 하기 위해서는 $elemMatch를 이용하여 검색 합니다. 

```` 
db.a.find({people : { $elemMatch : { firstname: "john", lastname:"doe"}}})
```` 
이 경우 배열내에서 firstname이 "john" 이고 lastname이 "doe"를 동시에 만족하는 데이터를 검색 하여 원하는 데이터를 참게 됩니다. 이를 이용하여 Array내에 nested document에 대해서도 원하는 검색 조건을 작성 할 수 있습니다.   


##### Array Handling $[identifier]
배열 내에서 $[]은 내부의 모든 데이터를 선택 하는 것입니다. 배열내의 데이터를 수정 할 때 모든 엘리먼트에 적용 하기 위해서는 다음과 같이 Query를 작성 합니다.   

```` 

{ <update operator>: { "<array>.$[]" : value } }
```` 

학생의 성적 데이터가 배열 데이터를 생성 합니다.

````
db = db.getSiblingDB("mql")
db.students.deleteMany({})
grades =  [
   { "_id" : 1, "grades" : [ 95, 92, 90 ] },
   { "_id" : 2, "grades" : [ 98, 100, 102 ] },
   { "_id" : 3, "grades" : [ 95, 110, 100 ] }
]
db.students.insertMany( grades )
```` 

모든 성적데이터(grades)에 대해 -10을 적용 하는 경우 다음과 같이 Query를 작성 합니다.  

```` 
db.students.updateMany(
   { },
   { $inc: { "grades.$[]" : -10 } }
)
````

그 결과를 확인 하면 배열내의 모든 데이터의 값이 10이 감소된 것을 확인 할 수 있습니다.
````
Atlas atlas-12kwtk-shard-0 [primary] mql> db.students.find()
[
  { _id: 1, grades: [ 85, 82, 80 ] },
  { _id: 2, grades: [ 88, 90, 92 ] },
  { _id: 3, grades: [ 85, 100, 90 ] }
]
````

배열의 데이터를 처리 할 때 조건을 추가 하여 검색 결과를 $[identifier]을 이용하여 식별 합니다.
검색 조건은 arrayFilters를 이용하여 검색 합니다. 

arrayFilters 옵션과 함께 사용 하여 연산자의 형태는 다음과 같습니다.

```` 
{ <update operator>: { "<array>.$[<identifier>]" : value } },
{ arrayFilters: [ { <identifier>: <condition> } ] }
```` 

다음과 같이 배열을 포함하는 데이터를 생성 하여 줍니다.

````
db = db.getSiblingDB("mql")
db.students.deleteMany({})
grades =  [
   { "_id" : 1, "grades" : [ 95, 92, 90 ] },
   { "_id" : 2, "grades" : [ 98, 100, 102 ] },
   { "_id" : 3, "grades" : [ 95, 110, 100 ] }
]
db.students.insertMany( grades )
```` 

grades 배열에서 100보다 크거나 같은 모든 요소를 업데이트 하려면 arrayFilters와 함께 필터링된 위치 연산자 $[identifier]를 사용 합니다.

```` 
db.students.updateMany(
   { },
   { $set: { "grades.$[element]" : 100 } },
   { arrayFilters: [ { "element": { $gte: 100 } } ] }
)
```` 

업데이트된 내용을 확인 하면 grades 내부에 100이 넘는 데이터는 100으로 수정된 것을 확인 할 수 있습니다.  

```` 
db.students.find()
[
  { _id: 1, grades: [ 95, 92, 90 ] },
  { _id: 2, grades: [ 98, 100, 100 ] },
  { _id: 3, grades: [ 95, 100, 100 ] }
]
```` 

배열내에 엘리먼트가 document형태인 경우 동일 하게 동작 합니다. 
과목별로 성적 데이터를 작성 합니다. 
```` 
db = db.getSiblingDB("mql")
db.students.deleteMany({})
subjectgrade = [{
      "_id" : 1,
      "grades" : [
         { "grade" : 80, "mean" : 75, "std" : 8 },
         { "grade" : 85, "mean" : 90, "std" : 6 },
         { "grade" : 85, "mean" : 85, "std" : 8 }
      ]
   },
   {
      "_id" : 2,
      "grades" : [
         { "grade" : 90, "mean" : 75, "std" : 8 },
         { "grade" : 87, "mean" : 90, "std" : 5 },
         { "grade" : 85, "mean" : 85, "std" : 6 }
      ]
   }
]
db.students.insertMany(subjectgrade)
```` 

데이터 중 grade가 85보다 큰 데이터 찾아 평균(mean)값을 100으로 수정 하는 Query를 작성 합니다.  
```` 
db.students.updateMany(
   { },
   { $set: { "grades.$[elem].mean" : 100 } },
   { arrayFilters: [ { "elem.grade": { $gte: 85 } } ] }
)
```` 

결과를 확인 하면 다음과 같이 mean 값이 수정된 것을 확인 할 수 있습니다. 
```` 
db.students.find()
[
  {
    _id: 1,
    grades: [
      { grade: 80, mean: 75, std: 8 },
      { grade: 85, mean: 100, std: 6 },
      { grade: 85, mean: 100, std: 8 }
    ]
  },
  {
    _id: 2,
    grades: [
      { grade: 90, mean: 100, std: 8 },
      { grade: 87, mean: 100, std: 5 },
      { grade: 85, mean: 100, std: 6 }
    ]
  }
]
```` 
배열을 $[identifier] 지정 하고 arrayFilter에서 identifier에 대해 상세한 조건을 입력 하여 배열 내의 엘리먼트에 대한 연산을 진행 할 수 있습니다.  


### Aggregation

##### Simple Pipeline - match, project, limit, sort, count

MQL의 find와 유사하게 작성 합니다. stage를 pipeline으로 연결 하는 형태가 다른점입니다. 구조적으로 작성이 되기 때문에 개발 및 디버깅이 편한 장점이 있습니다.  

$match 의 표현식은 다음과 같습니다.   

```` 
{ $match: { <expression>: <value> } }
```` 

expression은 비교 연산, regular표현, 논리연산 등이 가능 합니다.   

sample_mflix 의 movies 컬렉션에서 2010년에 상영된 영화화의 갯수를 검색   


```` 
db = db.getSiblingDB("sample_mflix")

match = {$match: {year:2010}}
count = {$count: "release_count"}
doc = db.movies.aggregate([match, count])

```` 
##### Simple Pipeline - match, project, limit, sort, count
Project를 이용하여 출력 대상 데이터를 조정 합니다. 

find에서 진행한 검색내용을 match, project, sort, limit 를 이용하여 작성 합니다.

project는 필드에 대해 include (1), exclude (0) 중 하나를 사용하여 표현 할 수 있습니다.   

sample_mflix 데이터베이스의 movies 컬렉션에서 1933년에 개봉한 영화 중 가장 많은 상을 수상한 영화를 반환하는 쿼리를 작성 하고, title과 award 내용을 출력 합니다.   

```` 
db = db.getSiblingDB("sample_mflix")

match = {$match:{year:1933}}
project = {$project:{_id:0,"title":1, "awards.wins":1}}
sort = {$sort:{"awards.wins":-1}}
limit = {$limit: 1}

doc = db.movies.aggregate([match, project, sort, limit])

```` 

그 결과는 다음과 같습니다.

```` 
[ { title: 'State Fair', awards: { wins: 4 } } ]

```` 

동일한 Stage를 반복하여 사용이 가능 합니다.   
genres 가 Drama 인 것 중에 2000년도에 발표된 영화를 검색 (2개의 match로 작성)


```` 
db = db.getSiblingDB("sample_mflix")

match1 = {$match:{genres:'Drama'}}
match2 = {$match:{year:2000}}
count = {$count: 'numberOfDrama'}

doc = db.movies.aggregate([match1,match2, count])

```` 

그 결과는 다음과 같이 345개가 출력 됩니다.
```` 
[ { numberOfDrama: 345 } ]
````


##### Aggregation option




##### 배열 처리


##### lookup - Join 이용하기


##### Group 만들기


##### 데이터 변경



##### 데이터 추출
