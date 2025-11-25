
# Preguntas de competencia

## QC1.
¿Qué tareas de prioridad alta con fecha límite antes del viernes 17:00 están pendientes?

### DL Query
```
Tarea 
  and tienePrioridad value Alta 
  and tieneEstado value Planificada 
  and tieneFechaLimite some xsd:dateTime[<= "2025-11-07T17:00:00-03:00"^^xsd:dateTime]
```
### Resultado esperado
*Estudio Ontología* (45m), límite 2025-11-07 17:00 debe aparecer.

## QC2.
¿Qué eventos se solapan con *Consulta Médica*?
### SparQL Query (versión CON OWL-Time)
```
PREFIX : <http://example.org/agenda#>
PREFIX time: <http://www.w3.org/2006/time#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT ?evento ?inicio ?fin
WHERE {
    ?evento a :Evento ;
            :ocurreEn ?int .
    ?int  time:hasBeginning ?inicio ;
            time:hasEnd     ?fin .
    
    ?inicio   time:inXSDDateTime ?inicioD.
    ?fin      time:inXSDDateTime ?finD.

    :ConsultaMedica :ocurreEn ?intCM .
    ?intCM  time:hasBeginning ?inicioCM ;
            time:hasEnd       ?finCM .
    
    ?inicioCM   time:inXSDDateTime ?inicioCM_D.
    ?finCM      time:inXSDDateTime ?finCM_D.

    FILTER (?evento != :ConsultaMedica)
    FILTER ( ?inicioD < ?finCM_D && ?finD > ?inicioCM_D )
}
```
### Resultado esperado
*Reunión de proyecto* 10:30–11:30, *Defensa Ing Onto* 11:15–11:30 y *Consulta Médica* 11:00–11:30 deben estar solapados.

## QC3.
Para una tarea de 45 minutos, ¿qué ventanas viables existen para el 03/11 de 10:00 a 11:15?

### SparQL Query
```
PREFIX time: <http://www.w3.org/2006/time#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX ag: <http://example.org/agenda#>

SELECT ?ventana ?inicio ?fin
WHERE {
    ?ventana a ag:VentanaDisponibilidad ;
             time:hasBeginning ?b ;
             time:hasEnd ?e ;
             time:hasXSDDuration ?v_duration .

    ?b time:inXSDDateTime ?inicio .
    ?e time:inXSDDateTime ?fin .

    ?participante ag:tieneVentana ?ventana .

    ?tarea a ag:Tarea ;
           ag:involucra ?participante ;
           ag:tieneDuracion ?duracionXSD .

    FILTER (?duracionXSD = ?v_duration)

    FILTER (?inicio >= "2025-11-03T10:00:00-03:00"^^xsd:dateTime)
    FILTER (?fin    <= "2025-11-03T11:30:00-03:00"^^xsd:dateTime)
}
```
### Resultado esperado
Hay una ventana de Gonzalo que sirve.

## QC6.
¿Qué tareas están bloqueadas por dependencias no satisfechas?

### SparQL Query
```
PREFIX : <http://example.org/agenda#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>

SELECT ?tarea ?dep
WHERE {
  ?tarea rdf:type :Tarea .
  ?tarea :dependeDe ?dep .

  OPTIONAL { ?dep :tieneEstado ?estado . }

  FILTER( !(?estado = :Completada) )
}
```
### Resultado esperado
EstudioOntologia, puesto que depende de LecturaPaper.

## QC9.
¿Qué tareas vencen en las próximas 48 h y no tienen slot asignado?

### SparQL Query
```
PREFIX : <http://example.org/agenda#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT ?tarea ?fecha
WHERE {
    ?tarea a :Tarea .
    ?tarea :tieneFechaLimite ?fecha .

    FILTER (
        ?fecha >= "2025-11-07T16:12:00-03:00"^^xsd:dateTime &&
        ?fecha <= "2025-11-07T18:22:00-03:00"^^xsd:dateTime
    )

    FILTER NOT EXISTS {
        ?tarea :tieneVentana ?v .
    }
}
```
### Resultado esperado
EstudioOntologia

## QC10.
¿Cuales son las actividades más prioritarias para aportar a la meta *RecibirseDeIngeniero*?

### SparQL Query (con Entailments)
```
PREFIX : <http://example.org/agenda#>

SELECT ?actividad ?prioridad
WHERE {
    ?actividad 
        :tienePrioridad ?prioridad ;
        :contribuyeA :RecibirseDeIngeniero .

    FILTER NOT EXISTS {
        ?otraActividad 
            :tienePrioridad ?prioridadOtra ;
            :contribuyeA :RecibirseDeIngeniero .

        FILTER (?otraActividad != ?actividad)

        # PRIORIDAD OTRA es estrictamente mayor (usamos camino transitivo)
        ?prioridadOtra :prioridadEsMayorQue+ ?prioridad .
    }
}
```
### Resultado esperado
DefensaIngOnto
