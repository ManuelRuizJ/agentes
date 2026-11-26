# Guía para entender y explicar el proyecto (presentación final)

## 0. Checklist para la entrega (todo lo demás ya está cerrado)

**Hoy en la noche**
- [x] **Cargar la laptop al 100 %.** El 24 de noviembre estaba en 25 % y con el voltaje bajo:
      sin cargador el LLM se apaga solo (a propósito) y la demo con CrewAI no corre con Ollama.
- [x] Con el cargador puesto, una sola prueba (~2 min): terminal 1 `./iniciar_ollama_cpu.sh`,
      terminal 2 `source venv/bin/activate && python main.py api`, terminal 3
      `python main.py crew A002 -p "quiero algo ligero y sin clases el viernes"` → debe salir la carga,
      "Validador: CONSISTENTE" y Shapley. Si sale, ya está.
- [x] Presentación: dejar **8 diapositivas** (portada, planteamiento, arquitectura,
      conocimiento, herramientas, desempeño, conclusiones, referencias), nombres y matrículas en la portada
      y exportar a PDF. Si usan el Canva: borrar Introducción, Justificación y Objetivos; borrar
      Metodología (o cambiarle 24–48 → **24–60**); en Herramientas los pesos deben quedar
      **0.30 avance · 0.25 horario · 0.25 carga · 0.20 preferencias**; UML y Trace/Shapley pueden
      quedar como respaldo después de Referencias, **fuera del PDF**.

**Ya verificado (24 nov)** — no hay que tocar nada:
- 52 pruebas pasan; repo en GitHub al día (último commit `fff1eee`).
- Página: 11 peticiones × 3 alumnos, todas aceptadas por HermiT (ver sección 12).
- Validar inscripción rechaza LIIN2104 con su axioma; auditoría y métricas registran todo.
- CrewAI en modo `--sin-llm` (plan B) funciona para el caso exitoso y el rechazo.
- La base de auditoría quedó **vacía**: la demo de mañana la llena. No hace falta borrarla.

**Mañana**: seguir la sección 7 tal cual.


---

## 1. El proyecto en 30 segundos

> "Hicimos un sistema que le recomienda a un alumno **qué materias inscribir**.
> En lugar de un solo programa que lo decida todo, hay **varios agentes
> especialistas** que colaboran: uno revisa el kárdex, tres compiten armando cargas,
> uno acomoda horarios, uno **valida con lógica formal** que no se rompa ninguna regla,
> y un coordinador elige la mejor opción y explica cuánto aportó cada agente."

Si solo recuerdan una frase:

> **"Los agentes proponen, la ontología pone las reglas y HermiT verifica que nadie las rompa."**

---

## 2. La analogía que lo explica todo: la oficina de inscripciones

Imaginen la oficina de inscripciones de la universidad, con un equipo de personas:

| En la oficina… | En nuestro sistema | Qué hace |
|---|---|---|
| El **reglamento oficial** de la universidad | **Ontología OWL** | Guarda el plan de estudios (79 materias), la seriación, los grupos y los alumnos. Son las reglas. |
| La persona que **revisa tu kárdex** | **AgenteHistorial** | Dice qué materias SÍ puedes cursar y cuáles no (porque ya las pasaste o te falta un prerrequisito). |
| **Tres asesores que compiten** | **3 agentes de Carga** (Avance, Equilibrio, Preferencias) | Cada uno arma su propuesta de materias con su propio criterio. |
| La persona de **horarios** | **AgenteHorarios** | Entiende lo que pediste ("en la mañana, sin viernes") y acomoda grupos sin choques. |
| El **jefe de la oficina** | **AgenteManager** | Organiza la competencia, califica todas las propuestas con la misma fórmula y elige la ganadora. |
| El **notario / árbitro** | **AgenteValidador + HermiT** | Revisa con lógica formal que la propuesta no viole el reglamento. Si lo viola, la frena. |
| El **archivo** donde queda registro de todo | **SQLite** | Guarda cada ejecución, quién hizo qué, cuánto tardó y los resultados. |
| La **ventanilla** donde se atienden solicitudes | **FastAPI** | La "puerta" por la que entran todas las peticiones (de la página web o de CrewAI). |
| El **gerente que coordina al equipo** | **CrewAI** | El marco que organiza a los agentes como un equipo con roles, metas y tareas. |
| La **evaluación de desempeño** del equipo | **Valores de Shapley** | Dice cuántos puntos de la recomendación final aportó cada agente. |

**Clave:** el LLM (llama3.2) **no decide nada importante**. Solo entiende lo que escribe el
alumno y redacta la explicación final. Las decisiones las toman reglas y lógica.

---

## 3. Una petición, de principio a fin (cuéntenlo como historia)

**Luis (A002)** escribe: *"quiero algo ligero, en la mañana y sin clases el viernes"*.

1. **AgenteHorarios entiende la petición** → carga ligera (~30 créditos), turno matutino, viernes libre.
2. **AgenteHistorial revisa su kárdex** → Luis reprobó Cálculo, así que esa tiene prioridad.
   Descarta las que ya aprobó y las que todavía no puede cursar por seriación.
3. **El Manager abre una "licitación" (Contract Net)** → les pide a los 3 agentes de Carga:
   "armen una carga de 24 a 60 créditos". Cada uno manda sus 2 mejores propuestas (6 en total).
4. **AgenteHorarios asigna grupos** a cada propuesta, sin choques y respetando el viernes libre.
5. **El Manager califica las 6** con la misma fórmula:
   `30% avance + 25% horario + 25% carga equilibrada + 20% preferencias`.
   Gana la de **CargaPreferencias** con **94.81/100**. Al ganador le dice *accept*, a los demás *reject*.
6. **El Validador revisa la ganadora con HermiT** → "la ontología sigue consistente" → **aprobada**.
7. **Se calculan los valores de Shapley** → Historial 41, Carga 28, Horarios 26 (suman 94.81).
8. **Todo se guarda en SQLite** y se muestra al alumno: 4 materias, 30 créditos, sin viernes.

---

## 4. Los conceptos difíciles, explicados fácil

### 4.1 ¿Qué es la ontología?
Es el **reglamento escrito de forma que una computadora pueda razonar con él**.
No es una base de datos normal: además de datos, tiene **reglas lógicas**.

- **TBox** = las reglas y los tipos de cosas ("toda Materia puede tener prerrequisitos").
- **ABox** = los datos concretos ("Cálculo es prerrequisito de Cálculo II", "Luis aprobó X").

Analogía: TBox es el reglamento; ABox es el expediente de cada alumno.

### 4.2 ¿Qué es HermiT?
Es un **razonador**: un programa que lee la ontología y saca conclusiones lógicas,
o descubre que algo **no puede ser verdad** (una *inconsistencia*).

Analogía: un juez que lee el reglamento y la solicitud y dice "esto contradice la regla 14".

### 4.3 ¿Cómo detecta HermiT que una materia no se puede inscribir?
Agregamos al reglamento **35 reglas de seriación** (una por cada materia seriada), del tipo:

> *"Toda propuesta que incluya **Proyecto de Innovación (LIIN2104)** solo puede ser para
> alumnos que hayan aprobado **Proyecto de Ingeniería IV (LIIB2104)**."*

Si Ana (A001) pide LIIN2104 y **no** ha aprobado LIIB2104, el reglamento dice "debió
aprobarla" y su kárdex dice "no la aprobó". Eso es una **contradicción**, y HermiT la detecta.

### 4.4 El "axioma de cierre" (la pregunta difícil que les pueden hacer)
En OWL rige el **mundo abierto**: si el kárdex no dice que Ana aprobó una materia, la
computadora **no asume que no la aprobó**, porque tal vez solo falta el dato.

Por eso, antes de razonar, le decimos explícitamente:
> *"Esta es la lista **COMPLETA** de lo que Ana aprobó. No hay más."*

Con eso sí puede concluir "no aprobó LIIB2104" y encontrar la contradicción.

Analogía: si te dicen "Ana aprobó Cálculo", no sabes si también aprobó Física. Pero si te
dicen "Ana aprobó **solo** Cálculo", ya sabes que Física no.

### 4.5 "Inferencia efímera en RAM" (reasoning offloading)
Cada vez que se valida algo:
1. Se hace una **copia del reglamento en memoria**.
2. Se escribe la solicitud **en la copia**.
3. HermiT revisa la copia.
4. **La copia se tira.**

El archivo original **nunca se modifica**, y una solicitud no "ensucia" a la siguiente.
Analogía: sacar una fotocopia, rayar la fotocopia y tirarla. El original queda limpio.

### 4.6 ¿Por qué SQLite **y** ontología? (el "modelo híbrido")
Cada una sirve para algo distinto:

| | Ontología + HermiT | SQLite |
|---|---|---|
| Sirve para | **Reglas y razonamiento** | **Guardar registros rápido** |
| Velocidad | ~0.8 segundos por validación | menos de 1 milisegundo |
| Ejemplo | "¿Esta carga rompe la seriación?" | "Guarda que a las 11:44 se recomendó esto" |

HermiT es **unas 960 veces más lento** que una escritura en SQLite. Por eso solo se usa
cuando hace falta validar, no para guardar cosas.

### 4.7 Contract Net (el protocolo de coordinación)
Es una **licitación**, como cuando una empresa pide cotizaciones:
1. **cfp** (*call for proposals*): el Manager anuncia "necesito una carga".
2. **propose**: cada agente de Carga manda su oferta.
3. **accept / reject**: el Manager acepta la mejor y rechaza las demás.
4. **inform**: el ganador confirma el resultado.

Esos nombres (cfp, propose, accept…) vienen del estándar **FIPA-ACL** para mensajes entre agentes.

### 4.8 Valores de Shapley (la parte de "auditoría")
Pregunta: *"Si la recomendación sacó 94.81 puntos, ¿cuántos puntos se le deben a cada agente?"*

Cómo se calcula, en palabras:
- Se prueba el sistema con **todas las combinaciones de agentes**: con los 3, solo con 2,
  solo con 1 y sin ninguno (son **8 combinaciones**).
- Se ve **cuánto sube la calificación cuando entra cada agente**, en todos los órdenes posibles.
- Se promedia. Ese promedio es su valor de Shapley (φ).

Propiedad importante: **la suma de los φ es igual a la calificación final** (41.05 + 27.82 + 25.94 = 94.81).

Analogía: tres personas hicieron un proyecto en equipo; Shapley reparte la calificación según
cuánto aportó cada quien de verdad, no según quién habló más.

Conclusión que sale de los datos: **AgenteHistorial es el que más aporta**, porque sin él se
proponen materias que el alumno no puede cursar.

### 4.9 CrewAI y las "@tool"
- **CrewAI** es un framework para formar un "equipo" de agentes con LLM. A cada uno se le da
  un **rol**, una **meta** y una **historia** (backstory).
- Una **@tool** es una herramienta que el agente puede usar. Las nuestras **llaman a la API**
  (con `httpx`, una librería para hacer peticiones web).
- Así, el LLM **no tiene las reglas en la cabeza**: solo decide "voy a usar la herramienta de
  historial", y la API le responde con datos verificados.

Hay 5 agentes en la Crew: Historial, Carga, Horarios, **Validador** y Coordinador.
Proceso **secuencial** (uno tras otro); también existe el **jerárquico** (el coordinador decide a quién asignar cada tarea).

### 4.10 ¿Por qué no usar solo un LLM (ChatGPT, llama)?
Lo medimos (`python main.py experimento`) con 20 alumnos de prueba:

| Sistema | Calificación promedio | Propuestas con materias inválidas |
|---|---|---|
| **Nuestro MAS** | **89.57** | **0 %** |
| LLM solo (llama3.2) | 36.10 | **100 %** |

El LLM solo **alucina**: recomienda materias que el alumno no puede cursar. Por eso en
nuestro sistema el LLM solo redacta, y además hay un **filtro anti-alucinación** que
descarta su texto si menciona materias que no están en la propuesta.

---

## 5. Diagrama simple para la diapositiva de arquitectura

```
 Alumno / Página web          CrewAI (5 agentes con LLM)
          │                            │  @tool + httpx
          └────────────┬───────────────┘
                       ▼
              ┌─────────────────┐
              │     FastAPI     │  ← la "ventanilla"
              └────────┬────────┘
       ┌───────────────┼────────────────┐
       ▼               ▼                ▼
  Ontología OWL    HermiT (RAM)       SQLite
  (reglamento)     (árbitro)          (archivo)
```

Cómo explicarlo en voz alta (15 segundos):
> "Las peticiones entran por FastAPI, ya sea desde la página o desde los agentes de CrewAI.
> FastAPI consulta la ontología para las reglas, le pide a HermiT que valide, y guarda todo
> en SQLite para poder auditarlo después."

---

## 6. Guion de las 8 diapositivas (10 minutos máximo)

Sugerencia de reparto: **Persona 1** = diapositivas 1–4 · **Persona 2** = diapositivas 5–8.
Cada una ~1 minuto, la 3 y la 6 un poco más.

**1 · Portada** (15 s) — Título, nombres, matrículas, programa. Solo leerla.

**2 · Problema y por qué agentes** (1 min)
> "Cada semestre el alumno arma su carga a mano: tiene que cuidar seriación, créditos,
> horarios y lo que le conviene. La plataforma valida requisitos, pero no recomienda.
> Un LLM solo no sirve: en nuestro experimento propuso materias inválidas el 100 % de las
> veces. Por eso dividimos el trabajo entre agentes especialistas y dejamos las reglas
> en una ontología."

**3 · Arquitectura en CrewAI** (1.5 min) — Mostrar la tabla de agentes (rol / meta) y el diagrama.
> "Son 5 agentes: Historial, Carga, Horarios, Validador y Coordinador. El proceso es
> secuencial. Cada agente tiene su herramienta, y esa herramienta llama a nuestra API."

**4 · Base de conocimiento** (1.5 min) — Ontología vs SQLite (tabla de la sección 4.6).
> "La ontología tiene el plan real de Ingeniería en Sistemas: 79 materias y 35 reglas de
> seriación. SQLite guarda las ejecuciones. Para validar, cargamos la ontología en memoria,
> le agregamos la solicitud, HermiT revisa si es consistente y la copia se descarta."

**5 · Herramientas y Contract Net** (1 min)
> "Las @tool de CrewAI usan httpx para llamar a FastAPI. Para coordinar usamos Contract Net:
> el Manager anuncia la tarea, tres agentes compiten con propuestas distintas y gana la
> mejor calificada con la misma fórmula para todos."

**6 · Desempeño y Shapley** (1.5 min) — Mostrar captura de la pestaña Auditoría.
> "Guardar en SQLite tarda menos de 1 ms; validar con HermiT, unos 0.8 s. La tasa de éxito
> fue del 100 %. Con Shapley medimos cuánto aporta cada agente: el Historial es el que
> más aporta porque evita materias inválidas. La suma de los valores da exactamente la
> calificación final."

**7 · Conclusiones y mejoras** (1 min)
> "Separar las reglas (ontología) del código hace que, si cambia el plan de estudios, solo
> se edite la ontología. El LLM no decide, solo redacta, y así no alucina.
> Mejoras: un modelo más rápido o con GPU, contenedores Docker, y RAG para consultar el
> reglamento completo de la universidad."

**8 · Referencias** — Solo mostrarla (W3C OWL 2, CrewAI, FastAPI, HermiT, Shapley 1953,
Smith 1980 Contract Net, libro del curso). Formato APA 7 o IEEE.

---

## 7. Guion de la demo en vivo (5 minutos)

### Antes de que les toque (15 min antes)
1. **Cargador conectado** (sin cargador el sistema apaga el LLM a propósito).
2. Terminal 1: `cd agente-carga3 && source venv/bin/activate && python main.py api`
3. Terminal 2: `./iniciar_ollama_cpu.sh`
4. Terminal 3: `cd agente-carga3 && source venv/bin/activate && python main.py diagnostico` → debe decir ✔.
5. La base de auditoría ya quedó vacía el 24 de nov. Solo si hiciste pruebas después y quieres limpiarla: `rm -f salidas/carga.db*` **con la API apagada** (antes del paso 2).
6. Abrir en el navegador `http://127.0.0.1:8077` y recargar (F5): la línea de estado de arriba debe decir que todo está listo.

### Minuto 1 — "Todo está funcionando"
- **Primero** lanzar en la terminal 3 el caso 1 (tarda ~2 min y mientras tanto se explica lo demás):
  `python main.py crew A002 -p "quiero algo ligero y sin clases el viernes"`
- Mostrar la página: la línea de estado de arriba (FastAPI, ontología con 79 materias y 35 axiomas, SQLite, HermiT).
> "El backend en FastAPI está arriba, la ontología está cargada y SQLite listo."

### Minuto 2 — Caso exitoso
- Mostrar la **terminal 1**: van apareciendo líneas como
  `[api] POST /agentes/historial ← crewai @tool consultar_historial_academico`.
> "Cada línea es un agente de CrewAI usando su herramienta, que llama a nuestra API."
- Cuando termine, mostrar en la terminal 3: la carga, **"Validador: CONSISTENTE"**, los Shapley
  y "auditoría SQLite → ejecución #N".

### Minuto 3 — Rechazo semántico (lo más importante de la demo)
- Abrir: `http://127.0.0.1:8077/#val=A001:LIIS2105,LIIS2112,LIIB2104,LICB2102,LIIN2104`
- Sale en **rojo**: RECHAZADA · HermiT: INCONSISTENTE · el axioma violado.
> "Ana pidió Proyecto de Innovación, pero no ha aprobado Proyecto de Ingeniería IV, que es
> su prerrequisito. HermiT encontró que la ontología se vuelve contradictoria y el Agente
> Validador frenó la inscripción. Esto no lo decide un if: lo demuestra el razonador."
- (Si sobra tiempo, la versión con CrewAI tarda ~1 min:
  `python main.py crew A001 -m LIIS2105,LIIS2112,LIIB2104,LICB2102,LIIN2104`)

### Minutos 4 y 5 — Auditoría
- Pestaña **Auditoría** → **Actualizar**.
- Señalar: tasa de éxito, **SQLite < 1 ms vs HermiT ~800 ms**, barras de Shapley.
- Clic en una fila de la tabla → se ven las **trazas** (qué agente hizo qué y cuánto tardó).
- Bajar a la **bitácora** y marcar **Mostrar solo las herramientas de CrewAI**: quedan solo las **@tool**.

### Plan B (si algo falla)
| Problema | Qué hacer |
|---|---|
| CrewAI se tarda demasiado | Ctrl+C y correr lo mismo con `--sin-llm` (mismas herramientas, segundos) |
| La página no responde | Revisar que la terminal 1 siga abierta |
| El LLM sale "apagado" | Conectar el cargador y recargar la página |

---

## 8. Preguntas que probablemente les hagan

**¿Por qué no usar solo ChatGPT / un LLM?**
> Porque alucina: en nuestro experimento propuso materias inválidas el 100 % de las veces.
> Aquí el LLM solo entiende el texto y redacta; las reglas las aplican la ontología y HermiT.

**¿Qué pasa si el LLM inventa algo en la explicación?**
> Hay un filtro: si menciona materias que no están en la propuesta, o contradice al validador,
> > su texto se descarta y se usa una plantilla.

**¿Por qué HermiT detecta el error y no un simple `if`?**
> Porque la regla vive en la ontología, no en el código. Si cambia el plan de estudios,
> se edita la ontología y el código no se toca. Eso es el "desacoplamiento" que pide la guía.

**¿Qué es el axioma de cierre y por qué lo necesitan?**
> OWL asume mundo abierto: que no diga "aprobó X" no significa que no la aprobó. El axioma
> de cierre dice "esta es la lista completa de lo que aprobó"; solo así HermiT puede
> concluir que le falta un prerrequisito.

**¿Por qué SQLite y no MongoDB?**
> La guía permite una base relacional. SQLite no necesita servidor ni internet, así que no
> puede fallar en la demo, y es transaccional: cada ejecución se guarda completa o no se guarda.

**¿Por qué la inferencia es "efímera"?**
> Para no modificar la ontología original y para que una solicitud no afecte a la siguiente.
> Se razona sobre una copia en memoria y se descarta.

**¿Qué significa que Shapley sume la calificación final?**
> Es la propiedad de "eficiencia": los puntos se reparten completos entre los agentes, sin
> sobrar ni faltar.

**¿Por qué tarda tanto CrewAI?**
> El LLM corre en CPU, sin tarjeta gráfica. La API responde en ~2 s; el resto es el modelo.
> Con GPU o un modelo en la nube bajaría a segundos.

**¿Secuencial o jerárquico?**
> Usamos secuencial porque es más predecible con un modelo pequeño. El jerárquico también
> está implementado (`--proceso jerarquico`).

**¿Usan Docker?**
> No. Todo corre directo en la máquina: el entorno virtual de Python (`venv`), Ollama con
> un script propio en el puerto 11435, Java para HermiT y SQLite, que es un solo archivo.
> Meterlo en contenedores es una de las mejoras que proponemos.

**¿Qué pasa si escribo algo que no entiende?**
> En la página se interpretan las peticiones con reglas (es instantáneo). Si no reconoce nada,
> usa el perfil guardado del alumno, nunca inventa. Con Ollama encendido, en la terminal, el
> LLM lee texto libre y lo convierte a JSON, y cada campo se valida antes de usarse.

**¿Los datos son reales?**
> El plan de estudios sí (79 materias del portal de la Ibero). Los alumnos, grupos y
> profesores son de prueba.

**¿Tienen pruebas?**
> Sí, 52 pruebas automáticas con pytest: ontología, reglas, Contract Net, Shapley,
> validador, API y las frases que entiende. Corren en ~20 segundos.

---

## 9. Números para memorizar

| Dato | Valor |
|---|---|
| Materias del plan (reales) | **79** |
| Axiomas de seriación | **35** |
| Agentes en el Contract Net | Historial, 3 de Carga, Horarios, Manager, Validador |
| Agentes en CrewAI | **5** |
| Rango de créditos | **24 a 60** |
| Fórmula del Manager | **30 / 25 / 25 / 20** (avance, horario, carga, preferencias) |
| Coaliciones para Shapley | **8** (2³) |
| Escritura en SQLite | **< 1 ms** |
| Validación con HermiT | **~0.8 s** (~960× más lento) |
| Recomendación completa por la API | **~1.3 s** |
| CrewAI caso exitoso / rechazo | **~2 min 15 s / ~1 min** (CPU) |
| MAS vs LLM solo | **89.6 vs 36.1** puntos; **0 % vs 100 %** materias inválidas |
| Pruebas automáticas | **52** |

---

## 10. Glosario rápido

- **MAS:** sistema multiagente.
- **Agente:** programa con un rol y una meta, que toma decisiones dentro de su especialidad.
- **Ontología OWL 2:** reglamento en un formato con el que una computadora puede razonar.
- **TBox / ABox:** reglas / datos concretos.
- **Razonador (HermiT):** programa que saca conclusiones de la ontología o detecta contradicciones.
- **Inconsistencia:** cuando los datos y las reglas no pueden ser verdad al mismo tiempo.
- **Contract Net:** protocolo de licitación entre agentes (cfp → propose → accept/reject → inform).
- **FIPA-ACL:** estándar de los tipos de mensaje entre agentes.
- **Shapley (φ):** cuántos puntos aportó cada agente al resultado.
- **CrewAI:** framework para organizar agentes con LLM como equipo.
- **@tool:** herramienta que un agente de CrewAI puede usar (aquí: llamar a la API).
- **httpx:** librería de Python para hacer peticiones web.
- **FastAPI:** framework para crear la API (la "ventanilla" del sistema).
- **SQLite:** base de datos relacional en un solo archivo.
- **Alucinación:** cuando un LLM inventa información.

---

## 11. Qué se entrega y cómo

1. **Presentación en PDF**: 8 diapositivas + referencias (Entregable A).
2. **Enlace al repo**: https://github.com/ManuelRuizJ/agente-carga-academica
   Es público y el README ya documenta todo (Entregable B).
3. Se sube a la plataforma del curso: el PDF + el enlace del repo.

---

## 12. Qué frases entiende el sistema (para la demo)

El cuadro "¿Qué prefiere el alumno?" se interpreta con reglas (y con el LLM si está encendido).
El rango institucional es **24 a 60 créditos** y está en la ontología.

| Si el alumno escribe… | El sistema entiende |
|---|---|
| "quiero llevar 60 créditos", "unos 50 créditos" | esa cifra exacta (se ajusta a 24–60 si se sale) |
| "la máxima cantidad de materias posibles", "lo más que se pueda", "todas las que pueda", "el máximo de créditos" | el **tope: 60 créditos**, hasta 10 materias |
| "carga pesada", "quiero adelantar", "muchas materias", "quiero terminar rápido la carrera", "me quiero graduar pronto" | carga pesada: **~48 créditos**, hasta 8 materias |
| "carga normal" (o nada) | ~40 créditos, hasta 7 materias |
| "carga ligera", "pocas materias, no me quiero saturar", "ando chambeando, poquitas", "trabajo medio tiempo", "no tengo mucho tiempo" | ~30 créditos, hasta 5 materias |
| "lo mínimo posible", "quiero llevar mínima", "lo menos posible" | el **mínimo: 24 créditos en 3 materias** (las de 8 créditos) |
| "quiero 3 materias", "máximo 5 materias" | limita el número de materias; si con ellas no se llega a la meta, la meta baja (3 materias → 24 créditos) |
| "quiero 2 materias" | con 2 no se llega a 24 créditos: el Manager sube a 3 y lo avisa |
| "en la tarde / en la mañana", "en línea / presencial / híbrida" | turno y modalidad |
| "sin clases el viernes" | ese día libre |
| "me interesa inteligencia artificial" | da prioridad a esa materia |

Si con la cifra pedida **todas** las propuestas chocan de horario, el Manager vuelve a licitar
con 4 créditos menos y lo avisa en "Lo que pidió" (ej.: A002 pide 60 → queda en 44 sin choques).
Frase recomendada para la demo: **A001 + "quiero llevar la máxima cantidad de materias posibles"**
→ 58 créditos, 9 materias, sin choques, aprobada por HermiT.
