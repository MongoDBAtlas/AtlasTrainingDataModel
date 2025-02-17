<img src="https://companieslogo.com/img/orig/MDB_BIG-ad812c6c.png?t=1648915248" width="50%" title="Github_Logo"/> <br>


# MongoDB Atlas Training Data Modeling & Advanced MQL

### MQL

#### Reviewing Query
기본적인 MQL을 이용한 데이터 조회를 진행 합니다.  
Mongosh을 이용하여 배포된 클러스터에 접속 후 다음을 실행 하여 줍니다.   
Aggregation을 진행 하기 전에 MQL의 find를 이용하여 데이터를 filter하고 데이터를 정렬 할 수 있습니다.  

##### 1. Basic MQL 작성

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


##### 2. $elemMatch를 이용

MQL의 구조는 단순하여 사용이 어렵지 않습니다. 이 중 유용한 기능은 $elemMatch 입니다.    
$elemMatch 연산자는 문서가 배열로 되어 있는 경우 쿼리하는데 유용하게 사용 할 수 있습니다.      

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

$elemMatch 의 사용방법은 다음과 같습니다.  

```` 
{ <field>: { $elemMatch: { <query1>, <query2>, ... } } }
```` 

firstname과 lastname 이 "john doe" 인 사람을 $elemMatch를 이용하여 검색 합니다.  

```` 
db.a.find({people : { $elemMatch : { firstname: "john", lastname:"doe"}}})
```` 
이 경우 배열내에서 firstname이 "john" 이고 lastname이 "doe"를 동시에 만족하는 데이터를 검색 하여 원하는 데이터를 참게 됩니다. 이를 이용하여 Array내에 nested document에 대해서도 원하는 검색 조건을 작성 할 수 있습니다.   


##### 3. Array Handling $[identifier]

배열로 되어 있는 필드에 대해 데이터를 수정 하기 위해서 배열에 대한 데이터를 지정하여 수정 하는 것이 필요 합니다.   
$[identifier]는 배열내의 데이터를 지정하여 데이터를 수정 할 수 있습니다.   
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

##### 1. Simple Pipeline - match, project, limit, sort, count

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

db.movies.aggregate([match, count])

```` 

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

db.movies.aggregate([match, project, sort, limit])

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

db.movies.aggregate([match1,match2, count])

```` 

그 결과는 다음과 같이 345개가 출력 됩니다.
```` 
[ { numberOfDrama: 345 } ]
````



##### 2. 배열 처리
기본 문제열을 엘리먼트로 한 배열에 대한 처리를 합니다.   
sample_mflix 데이터베이스의 movies 컬렉션에 영화 정보에 genres 필드에 장르 정보가 배열로 저장 되어 있습니다. 해당 내용 중 코미디와 드라마 장르의 영화를 검색 하여 봅니다.   
데이터 출력은 영화 제목(title), 장르(genres) 항목만을 출력 합니다.


```` 
db = db.getSiblingDB("sample_mflix")

match = {$match: {genres: {$all: ['Drama','Comedy']}}}
project = {$project:{title:1, genres:1, _id:0}}

db.movies.aggregate([match,project])

```` 
결과를 확인 하면 Comedy, Drama 두개가 포함된 영화만이 출력 됩니다.

all을 이용하여 두개 장르에 포함된 영화를 검색 하였으나 Drama 혹은 Comedy 인 내용을 검색 합니다.

```` 
db = db.getSiblingDB("sample_mflix")

match = {$match: {genres: {$in: ['Drama','Comedy']}}}
project = {$project:{title:1, genres:1, _id:0}}

db.movies.aggregate([match,project])

```` 
검색 내용을 확인 하면 장르 항목에 Drama 혹은 Comedy가 포함된 영화가 검색 됩니다.  

문서가 배열로 되어 있는 경우의 검색을 합니다.  
다음과 같이 데이터를 생성 하고 검색을 합니다.   

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

grade가 85 이고 mean이 90인 데이터를 검색 합니다. 

```` 
match = {$match:{'grades.grade':85, 'grades.mean':90}}

db.students.aggregate([match])
[
  {
    _id: 1,
    grades: [
      { grade: 80, mean: 75, std: 8 },
      { grade: 85, mean: 90, std: 6 },
      { grade: 85, mean: 85, std: 8 }
    ]
  },
  {
    _id: 2,
    grades: [
      { grade: 90, mean: 75, std: 8 },
      { grade: 87, mean: 90, std: 5 },
      { grade: 85, mean: 85, std: 6 }
    ]
  }
]
```` 
결과는 grade가 85 인것과 mean이 90인 것이 모두 검색 됩니다.

$elemMatch를 이용하여 검색을 합니다.

```` 
elemmatch = {$match: {grades:{"$elemMatch":{"grade":85,"mean":90}}}}

db.students.aggregate([elemmatch])
```` 

결과는 조회 조건에 만족하는 한건의 데이터가 조회 됩니다.

```` 
[
  {
    _id: 1,
    grades: [
      { grade: 80, mean: 75, std: 8 },
      { grade: 85, mean: 90, std: 6 },
      { grade: 85, mean: 85, std: 8 }
    ]
  }
]

```` 

추가 적인 방법으로 배열의 데이터를 개별로 분리 하여 처리를 하여 봅니다.  
배열로 되어 있는 내용에서 검색 하기 위해서는 unwind를 이용하여 검색 합니다.

$unwind의 사용 방법은 다음과 같습니다. 
includeArrayIndex를 지정하면 배열의 index를 추가 하며 perserveNullAndEmptyArrays를 이용하여 array 값이 null 혹은 empty있더라도 데이터를 리턴 하게 합니다.  
```` 
{
  $unwind:
    {
      path: <field path>,
      includeArrayIndex: <string>,
      preserveNullAndEmptyArrays: <boolean>
    }
}
```` 

unwind는 배열로 되어 있는 element를 개별로 분리 합니다. 
grade가 85이고 mean이 90인 검색 하기 위해 배열로 되어 있는 element를 분리 하고 검색 하여 줍니다.   

```` 
unwind={$unwind: "$grades"}

match = {$match:{'grades.grade':85, 'grades.mean':90}}

db.students.aggregate([unwind,match])
```` 
결과는 다음과 같이 출력 됩니다.

```` 
[ { _id: 1, grades: { grade: 85, mean: 90, std: 6 } } ]
```` 

##### 3. lookup - Join 이용하기

sample_mflix 데이터베이스에는 comments 컬렉션과 users 컬렉션이 있습니다.  
사용자가 생성한 코멘트에 대한 정보로 comments에 사용자 정보를 포함하고 있습니다. 
users의 데이터 중 이름이 "Mercedes Tyler"인 사람을 찾아 그가 게시한 Comments 를 찾습니다.   

해당 데이터를 검색 하면 다음과 같습니다.    

users 컬렉션       
````
{
  "_id": {
    "$oid": "59b99dedcfa9a34dcd78862d"
  },
  "name": "Mercedes Tyler",
  "email": "mercedes_tyler@fakegmail.com",
  "password": "$2b$12$ONDwIwR9NKF1Tp5GjGI12e8OFMxPELoFrk4x4Q3riJGWY6jl/UZAa"
}
````

comments 의 경우 다음과 같습니다.
````
[{
  "_id": {
    "$oid": "5a9427648b0beebeb69579e7"
  },
  "name": "Mercedes Tyler",
  "email": "mercedes_tyler@fakegmail.com",
  "movie_id": {
    "$oid": "573a1390f29313caabcd4323"
  },
  "text": "Eius veritatis vero facilis quaerat fuga temporibus. Praesentium expedita sequi repellat id. Corporis minima enim ex. Provident fugit nisi dignissimos nulla nam ipsum aliquam.",
  "date": {
    "$date": {
      "$numberLong": "1029646567000"
    }
  }
},
...
]
````

Lookup 의 사용 옵션은 다음과 같습니다. (let, pipeline은 5.0이상 에서 사용 가능 합니다.)  
````
{
   $lookup:
      {
         from: <foreign collection>,
         localField: <field from local collection's documents>,
         foreignField: <field from foreign collection's documents>,
         let: { <var_1>: <expression>, …, <var_n>: <expression> },
         pipeline: [ <pipeline to run> ],
         as: <output array field>
      }
}
````

SQL과 비교 하면 다음과 같습니다.

````
SELECT *, <output array field>
FROM localCollection
WHERE <output array field> IN (
   SELECT <documents as determined from the pipeline>
   FROM <foreignCollection>
   WHERE <foreignCollection.foreignField> = <localCollection.localField>
   AND <pipeline match condition>
);
````

Lookup을 이용하여 "Mercedes Tyler"가 작성한 Comment를 조회 합니다. user 컬렉션의 이메일과 comments에 이메일 정보가 같은 데이터 만을 조회 합니다.  
Lookup 내의 pipeline에 match를 사용하는 경우 $expr을 이용해야 합니다. 
출력은 사용자 이름과 이메일, 작성한 코멘트를 출력 하여 줍니다.  

````
db = db.getSiblingDB("mql")

match={$match:{name:"Mercedes Tyler"}}

lookup={$lookup:{from:"comments", let:{useremail:"$email"}, pipeline: [{$match:{$expr:{$eq:["$email","$$useremail"]}}},{$project:{text:1,email:1,date:1,_id:0}}], localField:"name",foreignField:"name", as:"Comments"}}

project= {$project:{_id:0, password:0}}

db.users.aggregate([match,lookup,project])
````

사용자 Mercede Tyler가 작성한 코멘트에 대한 정보가 Comments 필드에 배열로 출력 되는 것을 확인 할 수 있습니다.   

##### 4. Group 만들기
데이터를 그룹으로 집계하여 조회 합니다. 
영화 정보 컬렉션에서 2010년에 나온 영화의 갯수를 조회 하기 위해서는 데이터를 그룹화 하는 것이 필요 합니다.  
group 의 사용 방법은 다음과 같습니다.  
````
{
 $group:
   {
     _id: <expression>, // Group key
     <field1>: { <accumulator1> : <expression1> },
     ...
   }
}
````
SQL의 group by와 유사하며 출력되는 필드에 대해서는 accumulate가 필요 합니다.  

2010년에 나온 영화의 개수 조회는 다음과 같습니다. 

````
match={$match:{year:2010}}

group={$group:{_id:'$year',totalCount:{$sum:'$year'}}}
````
내부 필드에 대해서 데이터를 기준으로 한 데이터 집계를 사용 하여 봅니다.
영화 데이터를 출시 연도별로 집계 하고 runtime 에 대한 평균, 최대, 최소 시간을 출력 하여 줍니다.  

````
group={$group:{_id:'$year',avgRuntime:{$avg:'$runtime'}, maxRuntime:{$max:'$runtime'}, minRuntime:{$min:'$runtime'}}}
````

Aggregation을 이용한 데이터 집계는 SQL와 달리 집계되는 데이터를 배열 형태로 표시 할 수 있습니다. Acculate에 $addToSet을 이용하는 경우 그룹화된 데이터가 배열 형태로 출력이 됩니다. 
runtime 집계에 영화의 타입이 어떤 것이 있는지를 추가 하여 조회 합니다. 영화 type은 movie, series의 값이 있습니다. 

````
group={$group:{_id:'$year',avgRuntime:{$avg:'$runtime'}, maxRuntime:{$max:'$runtime'}, minRuntime:{$min:'$runtime'},genres:{$addToSet:'$type'}}}

db.movies.aggregate([group])
````

출력 결과를 확인 하면 데이터가 movie, series가 중복으로 들어가 있지 않는 형태로 배열 값으로 집계되는 것을 확인 할 수 있습니다.

데이터 집계시 값의 범위를 이용한 집계 방법도 제공 합니다. 
출시된 영화를 10년 단위로 집계 하여 봅니다. 
Bucket을 이용하여 1960~1969,1970~1979 형태로 출시 년도에 대한 집계 범위를 지정 하여 집계 합니다. 

bucket의 사용 방법은 다음과 같습니다.

````
{
  $bucket: {
      groupBy: <expression>,
      boundaries: [ <lowerbound1>, <lowerbound2>, ... ],
      default: <literal>,
      output: {
         <output1>: { <$accumulator expression> },
         ...
         <outputN>: { <$accumulator expression> }
      }
   }
}

````
조회 하기 위한 범위 조건을 boundaries에 입력 하여 줍니다.  

````
bucket = {$bucket:{groupBy:'$year', boundaries: [1900,1910,1920,1930,1940,1950,1960,1970,1980,1990,2000,2010,2020], default:'Other',output:{countOfMovies:{$sum:1},avgRuntime:{$avg:'$runtime'}, maxRuntime:{$max:'$runtime'}, minRuntime:{$min:'$runtime'},years:{$addToSet:'$year'}}}}

db.movies.aggregate([bucket])
````

결과를 확인 하면 10년 단위로 데이터를 그룹화한 것을 확인 할 수 있습니다.


##### 5. 뷰 생성 하기
MongoDB는 2가지 형태 뷰를 제공 합니다.    

Standard View    
데이터를 읽기 작업을 진행 할 때 뷰의 데이터가 계산 되어 별도의 디스크를 사용 하지 않습니다.    
Standard View는 기존 컬렉션의 인덱스를 사용하며 별도로 인덱스를 생성하거나 변경은 불가합니다.    

Ondemand View    
$merge, $out을 이용하여 데이터를 디스크에 저장하는 형태 입니다.    
별도의 디스크에 저장되기 때문에 인덱스를 생성 할 수 있으며 Standard view와 달리 데이터 조회시 연산이 필요 없기 때문에 읽기 성능이 좋습니다.   

Standard View를 이용하여 개인 정보를 포함하는 데이터를 제외하기 조회 하는 View를 생성 합니다.   

뷰를 생성하기 위한 방법은 다음과 같습니다.

````
db.createView ( <view> , <source> , <pipeline> ..)

````

Pipeline 항목에 뷰를 위한 처리 stage를 작성 하여 줍니다.  

뷰의 사용 사례 중 하나는 개인 정보를 포함하는 정보를 제외하고 조회 하거나 전체 데이터가 노출 되지 않도록 하여야 합니다.   
컬렉션에는 원본 데이터가 저장되어 있고 뷰를 이용하여 일부 민감 데이터에 대한 보안 처리를 할 수 있습니다.   

Sample_mflix 데이터베이스에 users 컬렉션에 사용자와 이메일 암호화된 비밀번호를 저장 하고 있습니다.   
이메일 주소의 아이디 일부분을 '***'로 가리고 이메일 도메인 만을 표시하여 개인 정보를 보호하는 뷰를 작성 합니다.    


````
db = db.getSiblingDB("sample_mflix")

emailMask = {'$project':{'email':{ $concat: [{ $substrCP: ["$email", 0, 2] }, "****@", { $arrayElemAt: [{ $split: ["$email", "@"] }, 1] }] }, name:1, password:1}}

db.createCollection('maskUsers',{'viewOn':'users',pipeline:[emailMask]})

db.maskUsers.find()

````

결과를 확인 하면 이메일 중 일부가 가려진 형태로 조회 되며 원본에 대한 조회 권한을 제안하여 개인 정보를 보호 합니다.

##### 6. Role을 이용한 뷰 생성 하기 (Option)

MongoDB 7.0부터 USER_ROLES 시스템 변수가 추가되어 이를 이용하여 데이터 접근에 대한 제어가 가능 합니다. (해당 기능은 Freetier에서 제공되지 않습니다.)    

users의 이메일 항목에 대해 접근 권한이 있는 경우 email 항목을 조회 할 수 있으며 그외에는 해당 항목 조회가 제한 됩니다.

email 필드에 대해 액세스 할 수 있는 PERSONAL_ACCESS 역할을 가지는 person 사용자

<img src="/01.MQL/images/image01.png" width="90%" height="90%">   

역할을 생성 합니다.

<img src="/01.MQL/images/image02.png" width="90%" height="90%">   

사용자를 생성 하여 줍니다. 이때 생성한 권한을 부여 하여 줍니다.

users 컬렉션에 PERSONAL_ACCESS 권한을 가진 경우만 email을 항목을 조회 할 수 있는 뷰를 생성 합니다.

````
emailMask = {$set: {"email": {$cond: { if:{$in:["PERSONAL_ACCESS","$$USER_ROLES.role"]}, then:"$email", else:"$$REMOVE"}}}}

db.createView('roleUser','users',[emailMask])

emailMask = {$set: {"email": {$cond: { if:{$in:["PERSONAL_ACCESS","$$USER_ROLES.role"]}, then:"$email", else:"$$REMOVE"}}}}
{
  '$set': {
    email: {
      '$cond': {
        if: { '$in': [ 'PERSONAL_ACCESS', '$$USER_ROLES.role' ] },
        then: '$email',
        else: '$$REMOVE'
      }
    }
  }
}

db.createView('roleUser','users',[emailMask])

db.roleUser.find().limit(5)


````

결과를 확인 하면 다음과 같이 email 항목이 제외 된것을 볼 수 있습니다.

````
[
  {
    _id: ObjectId("59b99db4cfa9a34dcd7885b6"),
    name: 'Ned Stark',
    password: '$2b$12$UREFwsRUoyF0CRqGNK0LzO0HM/jLhgUCNNIJ9RJAqMUQ74crlJ1Vu'
  },
  {
    _id: ObjectId("59b99db4cfa9a34dcd7885b7"),
    name: 'Robert Baratheon',
    password: '$2b$12$yGqxLG9LZpXA2xVDhuPnSOZd.VURVkz7wgOLY3pnO0s7u2S1ZO32y'
  },
  {
    _id: ObjectId("59b99db5cfa9a34dcd7885b8"),
    name: 'Jaime Lannister',
    password: '$2b$12$6vz7wiwO.EI5Rilvq1zUc./9480gb1uPtXcahDxIadgyC3PS8XCUK'
  },
  {
    _id: ObjectId("59b99db5cfa9a34dcd7885b9"),
    name: 'Catelyn Stark',
    password: '$2b$12$fiaTH5Sh1zKNFX2i/FTEreWGjxoJxvmV7XL.qlfqCr8CwOxK.mZWS'
  },
  {
    _id: ObjectId("59b99db6cfa9a34dcd7885ba"),
    name: 'Cersei Lannister',
    password: '$2b$12$FExjgr7CLhNCa.oUsB9seub8mqcHzkJCFZ8heMc8CeIKOZfeTKP8m'
  }
]

````

PERSONAL_ACCESS 권한을 가지고 있는 person 사용자로 변경하고 데이터를 조회 합니다.

````
use admin

db.auth('person')

use sample_mflix

db.roleUser.find().limit(5)

````

조회된 데이터를 확인 하면 다음과 같이 email이 포함되어 있는 것을 확인 할 수 있습니다.

````
[
  {
    _id: ObjectId("59b99db4cfa9a34dcd7885b6"),
    name: 'Ned Stark',
    email: 'sean_bean@gameofthron.es',
    password: '$2b$12$UREFwsRUoyF0CRqGNK0LzO0HM/jLhgUCNNIJ9RJAqMUQ74crlJ1Vu'
  },
  {
    _id: ObjectId("59b99db4cfa9a34dcd7885b7"),
    name: 'Robert Baratheon',
    email: 'mark_addy@gameofthron.es',
    password: '$2b$12$yGqxLG9LZpXA2xVDhuPnSOZd.VURVkz7wgOLY3pnO0s7u2S1ZO32y'
  },
  {
    _id: ObjectId("59b99db5cfa9a34dcd7885b8"),
    name: 'Jaime Lannister',
    email: 'nikolaj_coster-waldau@gameofthron.es',
    password: '$2b$12$6vz7wiwO.EI5Rilvq1zUc./9480gb1uPtXcahDxIadgyC3PS8XCUK'
  },
  {
    _id: ObjectId("59b99db5cfa9a34dcd7885b9"),
    name: 'Catelyn Stark',
    email: 'michelle_fairley@gameofthron.es',
    password: '$2b$12$fiaTH5Sh1zKNFX2i/FTEreWGjxoJxvmV7XL.qlfqCr8CwOxK.mZWS'
  },
  {
    _id: ObjectId("59b99db6cfa9a34dcd7885ba"),
    name: 'Cersei Lannister',
    email: 'lena_headey@gameofthron.es',
    password: '$2b$12$FExjgr7CLhNCa.oUsB9seub8mqcHzkJCFZ8heMc8CeIKOZfeTKP8m'
  }
]

````
