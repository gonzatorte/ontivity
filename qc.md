
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
*Estudio Ontología* (45′), límite 2025-11-07 17:00, estado *Planificada* debe aparecer.


## QC2.
¿Qué eventos se solapan con *Consulta Médica*?
### DL Query (versión CON OWL-Time)
```
Evento
  and ocurreEn value X
```
### DL Query (versión SIN OWL-Time)
```
Evento
  and ocurreEn value (
    ((inverseOf ocurreEn) 'Consulta Médica') finalizaEn > value X finalizaEn or
    ((inverseOf ocurreEn) 'Consulta Médica') iniciaEn < value X iniciaEn
  )
```
### Resultado esperado
*Reunión de proyecto* 10:30–11:30 y *Consulta Médica* 11:00–11:30 deben estar solapados.

## QC3.
Para una tarea flexible de 45′, ¿qué ventanas viables existen el martes 9:00–12:00 sin romper dependencias?

### DL Query (versión CON OWL-Time)
```
VentanaDeTiempo
P45M

Evento
  and ocurreEn value X
  and ocurreEn value ('has XSD duration' 'has Date-Time description')
  and X 'interval disjoint' (no-se-como-crear-el-intervalo "2025-11-07T17:09:00-03:00"^^xsd:dateTime to 2025-11-07T17:12:00-03:00"^^xsd:dateTime)
```
### DL Query (versión SIN OWL-Time)
```
Evento
  and ocurreEn value X
  and X iniciaEn "2025-11-07T17:09:00-03:00"^^xsd:dateTime
  and X finalizaEn "2025-11-07T17:12:00-03:00"^^xsd:dateTime
```
### Resultado esperado
Evidencia: no hay ex:Evento en el rango ∧ ¬(dependeDe no completada).

## QC6.
¿Qué tareas están bloqueadas por dependencias no satisfechas?

### DL Query
```
ex:dependeDe hacia actividad con ex:tieneEstado ≠ ex:Completada.
```
### Resultado esperado

## QC9.
¿Qué tareas vencen en las próximas 48 h y no tienen slot asignado?

### DL Query
```
tieneFechaLimite − ahora ≤ 48h ∧ ¬(ex:ocurreEn).
```
### Resultado esperado
