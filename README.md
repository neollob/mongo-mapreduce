# mongo-mapreduce

### Ejercicio __people__

1.  Cargar el fichero people.json en una bd personas
2. Generar por map-reduce una colección de personas (campo name) y su puntuación (campo points)

        var mapPeople = function(){
          emit({name:this.name}, this.points);
        };
        var reducePeople = function(dato,cuantos){
          return Array.sum(cuantos);
        };
        db.people.mapReduce(mapPeople,
                               reducePeople,
                               {out: "newPeople"}
        )
        
3. Comprobar el resultado sobre la nueva colección

        db.newPeople.find().pretty()
      
4. Generar el mismo resultado mediante una función aggregate

        db.people.aggregate(
        {$group:{_id:{"name":"$name"},total : {"$sum" :"$points"}}}
        )

### Ejercicio __movies__

Obtener por mapReduce una colección de directores y oscars de la colección movieDetails de la bd movies.

    mongoimport --db movies --collection movies --drop --file json\movieDetails.json
    
    var mapPeople = function(){
          emit({director:this.director}, this.awards);
        };
db.movies.aggregate(
        {$group:{_id:{"awards":"$awards.text"}}},
        {$match:{}}
        )
        
db.movies.find(
{ $and:
[
{"awards.text":{$in:[/^won/i]}},
{"awards.text":{$in:[/oscar*/i]}}
]
},{_id:0,"awards.text":1,"director":1}
).pretty()

var mapdirectores=function(){
  if (this.director){
    if (this.awards.text.indexOf(/\won \d+ oscar/i)>=0){
        this.awards.text.split(" ");
    
      emit({director:this.director.split(',')},this.awards);
    }
  }
}
var reddirectores=function(dato, valor){

 return dato;
 
}
db.movies.mapReduce(mapdirectores,
                               reddirectores,
                               {out: "newDirec"}
)
