# Guion de presentación: Agente de carga académica

Se sigue el README de arriba hacia abajo. **Muestra** = qué parte del README enseñar ·
**Teclea** = comando en la terminal · **Señala** = qué mostrar en la salida.
Duración total: ≈10 min.

---

## Antes de presentar (10 min antes)

1. Conecta el cargador (sin cargador el LLM se desactiva solo).
2. **Terminal 1:** `./iniciar_ollama_cpu.sh` y déjala abierta.
3. **Terminal 2:** `cd agente-carga3` y `source venv/bin/activate`
4. Corre una vez los dos `recomendar` con LLM que vas a enseñar, para que queden en caché
   y en la presentación salgan en menos de 1 s. Si preguntan: "la primera vez tarda 30–60 s
   en CPU; las respuestas se guardan en caché".
5. Compara que lo que salga coincida con los "Resultado" del README.
6. **No** corras `crew` ni `experimento --llm` en vivo (tardan minutos): solo se mencionan.

---

## Parte 1 · Título (README, inicio) · ≈30 s

> "Nuestro proyecto es un sistema multiagente que recomienda a un alumno qué materias
> inscribir en el periodo. Respeta la seriación, el rango de 24 a 60 créditos y que no
> haya choques de horario, y además toma en cuenta lo que el alumno quiere: turno,
> modalidad, días libres e intereses.
> La plataforma de inscripción valida requisitos, pero no te dice qué combinación te
> conviene. Eso es lo que resolvemos."

---

## Parte 2 · Cómo funciona: el diagrama · ≈1 min

**Muestra:** el diagrama; recórrelo de arriba hacia abajo con el cursor.

> "Empieza con la **petición del alumno**, en lenguaje natural, por ejemplo 'quiero algo
> ligero y sin clases el viernes'. El **Agente de Horarios** la interpreta y la convierte
> en preferencias.
> Luego el **Manager** coordina todo con el protocolo **Contract Net**. Primero manda un
> `cfp`, *call for proposals*, o sea, 'mándenme sus propuestas':
> - al **Agente de Historial**, que regresa las materias válidas;
> - a los **3 agentes de Carga**, que **compiten**, cada uno con su oferta;
> - al **Agente de Horarios**, que asigna grupos sin choques.
>
> Después el Manager **evalúa** las ofertas, **acepta** la mejor y **rechaza** las demás,
> y el ganador **confirma** con un `inform`. Al final calcula los **valores de Shapley**.
> Todo se lee y se guarda en la **ontología OWL**."

---

## Parte 3 · Tabla de agentes · ≈1 min

> "Son **4 roles y 6 agentes**, porque el rol de Carga lo cubren 3:
> - **Historial** lee el kárdex y descarta lo que no puedes cursar: lo que ya aprobaste,
>   lo que no tiene su prerrequisito y lo que no se oferta. Da prioridad a las reprobadas
>   y a las materias 'llave', que son prerrequisito de varias.
> - **Carga** son tres contratistas con estrategias distintas: **Avance** prioriza avanzar
>   en la carrera, **Equilibrio** busca la meta de créditos con pocas materias difíciles y
>   **Preferencias** se fija en intereses, turno y modalidad. Cada uno manda sus 2 mejores
>   ofertas.
> - **Horarios** interpreta la petición y asigna grupos sin choques.
> - **Manager** coordina y califica a todos con la misma función."

---

## Parte 4 · Función, Shapley, ontología y LLM · ≈1.5 min

**Función de evaluación:**
> "Todas las ofertas se califican con la misma función: 30 % avance académico, 25 %
> horario, 25 % equilibrio de carga y 20 % preferencias. Es importante que sea la misma,
> porque cada contratista se autoevalúa con su propia escala y esas autoevaluaciones no se
> pueden comparar."

**Transparencia (Shapley):**
> "Para que el sistema sea transparente usamos el **valor de Shapley**, de teoría de
> juegos. Probamos todas las combinaciones de agentes; cuando un agente no está, su tarea
> se hace de forma 'ingenua'. Así medimos cuántos puntos aportó cada uno. Con 3 agentes
> son 8 combinaciones, así que el cálculo es exacto, y la suma de los aportes da la
> puntuación final."

**Base de conocimiento:**
> "La base de conocimiento es una **ontología OWL** con el **plan real** de Sistemas
> Computacionales: 79 materias con su seriación. Los agentes la consultan con **SPARQL** y
> escriben ahí sus resultados. El razonador **HermiT** deduce clases solo; por ejemplo, las
> materias llave son las que son prerrequisito de al menos dos materias."

**LLM:**
> "El LLM es **Llama 3.2** con Ollama. Solo **interpreta** la petición y **redacta** la
> explicación; **no decide** qué materias son válidas, eso lo deciden las reglas y la
> ontología. Si el LLM no está, el sistema usa reglas y da la misma recomendación. El plan
> es real; los alumnos y los grupos son datos de prueba."

---

## Parte 5 · Demo sin LLM · ≈3 min

**Teclea:** `python main.py alumnos`
**Muestra:** la tabla "Resultado" debajo del comando.
> "Tenemos 3 alumnos de prueba: Ana va en 5.º y reprobó Programación Avanzada; Luis va en
> 3.º y reprobó Cálculo; Sofía va en 6.º y no debe nada."

**Teclea:** `python main.py recomendar A001 --sin-llm`
**Señala en la terminal**, en este orden:
1. **`cfp` / `propose`:** "Aquí se ve el Contract Net: el Manager pregunta y cada agente
   propone."
2. **`accept-proposal` / `reject-proposal`:** "Aceptó la oferta de CargaPreferencias con
   92.29 y rechazó las demás."
3. Luego pasa a la **tabla del README:** "6 materias, 40 créditos, con grupo y horario, sin
   choques. Pone primero Programación Avanzada porque la reprobó y es prerrequisito de
   otras."
4. **Licitación:** "Los tres compitieron; ganó Preferencias."
5. **Shapley** (en la terminal): "Historial aportó 39.4 puntos, Carga 30.1 y Horarios 22.8.
   Suman 92.3, la puntuación final."

**Teclea:** `python main.py demo` (opcional, si quieres mostrar los 3 alumnos)
**Muestra:** la tabla del README.
> "En Ana y Luis, que reprobaron materias, el que más aporta es Historial, porque la
> seriación descarta muchas. En Sofía, que no debe nada y pide carga pesada, aporta más
> Carga."

**Teclea:** `python main.py experimento` (≈4 s)
**Muestra:** la tabla del README.
> "Para saber si el multiagente de verdad sirve, lo comparamos con 20 alumnos de prueba.
> El MAS saca 89.6 y nunca propone materias inválidas ni choques. Un monoagente que solo
> revisa requisitos saca 70.9. Un LLM solo, al que le dimos el plan y el kárdex, saca 36.1
> y propone materias inválidas en todos los casos. Por eso el LLM no decide."

---

## Parte 6 · Con LLM · ≈2 min

**Muestra:** los pasos 1–3.
> "Para usar el LLM primero se levanta Ollama en una terminal, solo con CPU, y se deja
> abierta." (Señala la terminal 1.)

**Teclea:** `python main.py diagnostico`
**Señala:** "Ollama responde y el cargador está conectado."

**Teclea:**
`python main.py recomendar A002 -p "quiero algo ligero, en la mañana y sin clases el viernes"`
**Señala:**
> "Ahora la petición la entendió el LLM: carga ligera, turno matutino y sin viernes.
> Recomienda 4 materias y 30 créditos, recursa Cálculo y no pone ninguna clase el viernes.
> La explicación del final la redactó el LLM."

**Opcional:** `python main.py recomendar A001 -p "me interesa inteligencia artificial, prefiero la tarde" --razonador`
> "Ahora incluye Fundamentos de IA. Con `--razonador`, HermiT deduce las materias llave y
> los alumnos con adeudo."

**Sin teclear, señala `crew` y `experimento --llm`:**
> "También se puede orquestar con **CrewAI**, donde el LLM llama a las herramientas de
> cada agente. La decisión es la misma, pero en CPU tarda varios minutos, así que no lo
> corremos en vivo. Lo mismo el experimento que incluye al LLM solo."

**Párrafo final de Uso:**
> "En el reporte se ve si se usó el LLM o el respaldo con reglas, y las respuestas se
> guardan en caché."

---

## Parte 7 · Resultados, estructura y cierre · ≈1 min

> "Cada ejecución deja en `salidas/` un **reporte** con la recomendación, el horario
> semanal, las ofertas y el Shapley; las **trazas** de cada agente; y la **ontología con
> los resultados**, que se puede abrir en Protégé. También está el notebook, que muestra el
> proceso paso a paso."

(Si sobra tiempo: abre `salidas/reporte_A001.md` y enseña el **horario semanal**.)

> "En el código, cada parte tiene su archivo: `agentes.py` los agentes, `contract_net.py`
> el protocolo y Shapley, `kb.py` el acceso a la ontología y `evaluacion.py` las reglas y
> la función."

**Cierre:**
> "En resumen: dividir el problema entre especialistas da recomendaciones siempre válidas,
> la ontología permite cambiar el plan y las reglas sin tocar el código y Shapley explica
> cuánto aportó cada agente. Como trabajo futuro queremos usar el kárdex y la oferta reales
> y probarlo con alumnos."

---

## Datos para memorizar

| Dato | Valor |
|---|---|
| Roles / agentes | 4 roles, 6 agentes (Carga = 3 contratistas) |
| Función | 0.30 Avance + 0.25 Horario + 0.25 Carga + 0.20 Preferencias |
| Reglas | 24–60 créditos, máximo 2 optativas, sin choques |
| Ontología | 36 clases, 57 object properties, 50 data properties |
| Datos | 79 materias reales, 97 grupos, 452 individuos |
| Materias llave | Programación Avanzada y Algoritmos, Bases de Datos I, Persona y Humanismo |
| LLM | Llama 3.2 (3B), en CPU, 2 llamadas (interpretar y redactar) |
| Shapley | 3 jugadores (A, B, C), 8 coaliciones, exacto |
| Alumnos | A001 = 92.29 · A002 = 94.53 · A003 = 84.39 |
| Experimento | MAS 89.57 · un agente de Carga 88.90 · monoagente 70.90 · LLM solo 36.10 |

---

## Shapley con un ejemplo (A001)

v(A) = 76.56 · v(B) = 66.58 · v(C) = 63.25 · v(AB) = 87.77 · v(AC) = 76.56 ·
v(BC) = 67.86 · v(ABC) = 92.29

Aporte de Historial (A): promedio de cuánto suma A al entrar a cada coalición.

| A entra… | Suma | Peso |
|---|---|---|
| solo | 76.56 − 0 = 76.56 | 1/3 |
| con B | 87.77 − 66.58 = 21.19 | 1/6 |
| con C | 76.56 − 63.25 = 13.31 | 1/6 |
| con B y C | 92.29 − 67.86 = 24.43 | 1/3 |

**φ_A = 25.52 + 3.53 + 2.22 + 8.14 = 39.41**

En una frase: *"Shapley es el promedio de cuánto suma cada agente al entrar en todos los
órdenes posibles."*

---

## Preguntas probables

**¿Cuántos agentes son?**
4 roles y 6 agentes: Historial, 3 de Carga, Horarios y Manager.

**¿Qué es `cfp`?**
*Call for proposals*: el anuncio del Contract Net. Luego vienen `propose`,
`accept`/`reject-proposal` e `inform` (mensajes del estándar FIPA-ACL).

**¿Por qué no un solo agente?**
El monoagente sacó 18.7 puntos menos y el 15 % de sus cargas tenía choques. Además, con
varios agentes se puede medir quién aportó qué.

**¿Por qué el LLM no decide?**
Porque inventa: el LLM solo propuso materias inválidas en el 100 % de los casos. Las reglas
formales garantizan que la carga sea válida.

**¿Qué es "ingenuo" en Shapley?**
Lo que pasa cuando falta un agente. Sin Historial, todo lo ofertado parece válido. Sin
Carga, se toman 5 materias en orden. Sin Horarios, se toma el primer grupo de cada materia
sin revisar choques.

**¿Qué es el mundo abierto (OWA)?**
OWL no supone que lo que no está escrito sea falso: que no haya un registro de "aprobada"
no significa que no la aprobó. Por eso el Agente de Historial revisa el kárdex completo.
Por lo mismo se declara *AllDifferent* entre materias: OWL no asume que dos nombres
distintos sean cosas distintas.

**¿Para qué 3 contratistas si la mejora es pequeña?**
Suben el promedio de 0.5 a 1.1 puntos y, sobre todo, el peor caso (de 77.66 a 84.06).
Además respetaron los 2 días libres pedidos; el agente único, solo 1.

**¿Por qué el Manager no usa la autoevaluación de los contratistas?**
Cada uno se califica con su propia escala; no se pueden comparar entre sí.

**¿Qué propiedades de Shapley se ven?**
Eficiencia: la suma de los aportes da la puntuación final. Simetría: en A003 los 3
contratistas ofertaron lo mismo y cada uno recibió 10.2.

**¿De dónde salen los pesos de la función?**
Los definimos nosotros. Es una limitación; se plantea ajustarlos con datos de uso.

**¿Qué es real y qué es de prueba?**
El plan de estudios es real (exportado de Intrauia). Los alumnos, los grupos y los
profesores son de prueba.

**¿Qué hace CrewAI?**
Es otra forma de orquestar a los mismos agentes, donde el LLM llama a las herramientas.
La lógica de decisión es la misma.

**¿Qué pasa si Ollama no está?**
El sistema interpreta la petición con reglas y usa una plantilla para la explicación. La
recomendación es la misma.
