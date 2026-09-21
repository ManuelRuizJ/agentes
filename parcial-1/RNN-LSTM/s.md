# RNN & LSTM — Presentation Speech

---

# SLIDE 1 — TITLE

## 🇺🇸 English

Good morning. Today we are going to talk about Recurrent Neural Networks, also known as RNNs, and Long Short-Term Memory networks, or LSTMs, for sequence processing.

The main idea behind these architectures is to process data where the order of the elements matters.

Unlike traditional feed-forward neural networks, RNNs can use information from previous time steps when processing the current input.

We will explain why sequence processing is important, how RNNs and LSTMs work, their activation functions, and finally how these models are trained and optimized.

## 🇲🇽 Español

Buenos días. Hoy vamos a hablar sobre las Redes Neuronales Recurrentes, conocidas como RNNs, y las redes Long Short-Term Memory, o LSTMs, para el procesamiento de secuencias.

La idea principal de estas arquitecturas es procesar datos donde el orden de los elementos es importante.

A diferencia de las redes neuronales feed-forward tradicionales, las RNN pueden utilizar información de pasos anteriores al procesar la entrada actual.

Vamos a explicar por qué es importante procesar secuencias, cómo funcionan las RNN y LSTM, sus funciones de activación y finalmente cómo se entrenan y optimizan.

---

# SLIDE 2 — BACKGROUND

## 🇺🇸 English

To understand why RNNs are useful, we first need to understand the limitation of traditional feed-forward neural networks.

In a feed-forward network, information moves from the input to the output without explicitly maintaining information from previous inputs.

However, many real-world problems are sequential.

For example, when processing a sentence, the meaning of a word can depend on words that appeared before it.

RNNs fix this problem by maintaining a hidden state that is updated as the sequence is processed.

This gives the network a representation of information from previous time steps.

## 🇲🇽 Español

Para entender por qué las RNN son útiles, primero tenemos que entender la limitación de las redes feed-forward tradicionales.

En una red feed-forward, la información va desde la entrada hasta la salida sin mantener explícitamente información de entradas anteriores.

Sin embargo, muchos problemas del mundo real son secuenciales.

Por ejemplo, al procesar una oración, el significado de una palabra puede depender de palabras que aparecieron anteriormente.

Las RNN abordan este problema manteniendo un estado oculto que se actualiza conforme se procesa la secuencia.

Esto le permite a la red representar información de pasos anteriores.

---

# SLIDE 3 — WHAT PROBLEMS CAN RNNs AND LSTMs SOLVE?

## 🇺🇸 English

Because RNNs are designed for sequential information, they can be applied to many different types of problems.

In natural language processing, sequences can be words or tokens.

In speech processing, the sequence can be audio samples or features extracted over time.

For time-series forecasting, the sequence can contain values such as temperature, energy consumption, or financial measurements over different time steps.

They can also be used for sensor data, music and signal processing, and general sequence classification.

The common characteristic is that the order and temporal relationship between observations can contain useful information.

## 🇲🇽 Español

Debido a que las RNN están diseñadas para información secuencial, pueden aplicarse a diferentes tipos de problemas.

En procesamiento de lenguaje natural, las secuencias pueden ser palabras o tokens.

En procesamiento de voz, la secuencia puede estar formada por muestras de audio o características extraídas a través del tiempo.

En series temporales, la secuencia puede contener valores como temperatura, consumo de energía o mediciones financieras en diferentes momentos.

También pueden utilizarse para datos de sensores, procesamiento de música y señales, y clasificación de secuencias.

La característica común es que el orden y la relación temporal entre las observaciones pueden contener información útil.

---

# SLIDE 4 — RNN ARCHITECTURE

## 🇺🇸 English

Now we can look at the basic architecture of an RNN.

At each time step, the network receives two important pieces of information: the current input, represented by x sub t, and the previous hidden state, represented by h sub t minus one.

These two values are combined and passed through a function that produces the new hidden state, h sub t.

This new hidden state can be used to produce an output and is also passed to the next time step.

This recurrence is what gives the architecture its name and allows information to flow through the sequence.

The basic equation can be written as:

h sub t equals tanh of W sub x h times x sub t, plus W sub h h times h sub t minus one, plus the bias.

## 🇲🇽 Español

Ahora podemos observar la arquitectura básica de una RNN.

En cada paso temporal, la red recibe dos elementos importantes: la entrada actual, representada por x sub t, y el estado oculto anterior, representado por h sub t menos uno.

Estos dos valores se combinan y pasan por una función que produce el nuevo estado oculto, h sub t.

Este nuevo estado oculto puede utilizarse para producir una salida y también se pasa al siguiente paso temporal.

Esta recurrencia es lo que le da su nombre a la arquitectura y permite que la información fluya a través de la secuencia.

La ecuación básica puede escribirse como:

h sub t es igual a tanh de W sub x h por x sub t, más W sub h h por h sub t menos uno, más el bias.

---

# SLIDE 5 — LONG SHORT-TERM MEMORY

## 🇺🇸 English

The main limitation of basic RNNs is that they can have difficulty learning long-term dependencies.

As sequences become longer, information from earlier time steps can become difficult to preserve during training.

LSTM networks were designed to resolve this problem.

An LSTM introduces a cell state and gates that control how information is stored, forgotten, and exposed as output.

The three main gates are the forget gate, the input gate, and the output gate.

The forget gate determines what information should be removed from the cell state.

The input gate controls what new information should be stored.

The output gate determines what information from the internal state should be exposed as the hidden state.

There is also a candidate memory, which represents new information that could be added to the cell state.

## 🇲🇽 Español

La principal limitación de las RNN básicas es que pueden tener dificultades para aprender dependencias de largo plazo.

Conforme las secuencias se hacen más largas, la información de pasos anteriores puede ser difícil de conservar durante el entrenamiento.

Las redes LSTM fueron diseñadas para abordar este problema.

Una LSTM introduce un estado de celda y diferentes puertas que controlan cómo se almacena, olvida y expone la información.

Las tres puertas principales son la forget gate, la input gate y la output gate.

La forget gate determina qué información debe eliminarse del estado de celda.

La input gate controla qué información nueva debe almacenarse.

La output gate determina qué información del estado interno debe exponerse como hidden state.

También existe un candidate memory, que representa información nueva que podría agregarse al estado de celda.

---

# SLIDE 6 — ACTIVATION FUNCTIONS & NON-LINEARITY

## 🇺🇸 English

Activation functions introduce non-linearity into neural networks, allowing them to learn more complex relationships.

In a standard RNN, Tanh is commonly used to compute the hidden state. It produces values between minus one and one.

LSTMs use both Tanh and Sigmoid.

Sigmoid is especially useful for the gates because it produces values between zero and one. This allows the gates to control how much information should pass through.

(descartar) ReLU is another widely used activation function in neural networks, although it is not the main activation used inside the LSTM gates. -

Finally, Softmax is commonly used at the output of a classification model to convert logits into a probability distribution across the classes.

## 🇲🇽 Español

Las funciones de activación introducen no linealidad en las redes neuronales, permitiendo que aprendan relaciones más complejas.

En una RNN estándar, Tanh se utiliza comúnmente para calcular el estado oculto. Produce valores entre menos uno y uno.

Las LSTM utilizan tanto Tanh como Sigmoid.

Sigmoid es especialmente útil para las puertas porque produce valores entre cero y uno. Esto permite que las puertas controlen cuánta información debe pasar.

ReLU es otra función de activación muy utilizada en redes neuronales, aunque no es la activación principal utilizada dentro de las puertas de una LSTM.

Finalmente, Softmax se utiliza comúnmente en la salida de un modelo de clasificación para convertir los logits en una distribución de probabilidades entre las clases.

---

# SLIDE 7 — LEARNING AND OPTIMIZATION

## 🇺🇸 English

Once the architecture is defined, the model needs to learn its parameters from data.

First, during forward propagation, the input sequence passes through the network and produces a prediction.

The prediction is compared with the correct target using a loss function.

The loss measures how different the prediction is from the desired output.

Then, backpropagation calculates how the loss changes with respect to the model parameters.

For recurrent networks, this process is called Backpropagation Through Time, or BPTT, because the recurrent network is unfolded across the time steps of the sequence.

Finally, an optimization algorithm such as Gradient Descent or Adam uses these gradients to update the weights and reduce the loss.

This process is repeated over many times.

## 🇲🇽 Español

Una vez definida la arquitectura, el modelo necesita aprender sus parámetros a partir de los datos.

Primero, durante el forward propagation, la secuencia de entrada pasa por la red y produce una predicción.

La predicción se compara con el resultado correcto mediante una función de pérdida.

La pérdida mide qué tan diferente es la predicción respecto al resultado esperado.

Después, backpropagation calcula cómo cambia la pérdida respecto a los parámetros del modelo.

En redes recurrentes, este proceso se conoce como Backpropagation Through Time, o BPTT, porque la red recurrente se desdobla a través de los diferentes pasos temporales de la secuencia.

Finalmente, un algoritmo de optimización como Gradient Descent o Adam utiliza los gradientes para actualizar los pesos y reducir la pérdida.

Este proceso se repite durante múltiples épocas.

---

# SLIDE 8 — LSTM ARCHITECTURE

## 🇺🇸 English

This diagram shows how an LSTM processes an entire sequence.

The sequence is divided into individual time steps, and each LSTM cell receives the current input together with information from the previous time step.

Inside each cell, the gates determine how the cell state is updated.

The cell state carries information through the sequence, while the hidden state represents the information exposed by the LSTM at each time step.

This mechanism allows the network to selectively preserve relevant information and discard information that is no longer useful.

For a sequence classification problem, the information from the final hidden state can be passed to a final layer that produces the prediction.

## 🇲🇽 Español

Este diagrama muestra cómo una LSTM procesa una secuencia completa.

La secuencia se divide en diferentes pasos temporales y cada célula LSTM recibe la entrada actual junto con información del paso anterior.

Dentro de cada célula, las puertas determinan cómo se actualiza el estado de celda.

El estado de celda transporta información a través de la secuencia, mientras que el estado oculto representa la información que la LSTM expone en cada paso temporal.

Este mecanismo permite conservar selectivamente información relevante y descartar información que ya no es útil.

En un problema de clasificación de secuencias, la información del último estado oculto puede pasar a una capa final que produce la predicción.

---

# SLIDE 9 — IMPLEMENTATION

## 🇺🇸 English

For the practical implementation, we used Python and PyTorch to build and train recurrent models for sequence classification.

We used the handwritten digits dataset from scikit-learn.

Each image has a size of 8 by 8 pixels.

For this experiment, we intentionally interpreted each image as a sequence of eight rows, with eight pixel values as features at each time step.

The RNN and LSTM process those eight time steps sequentially.

The final hidden state is then used as a summary of the complete sequence and is passed to a linear classification layer.

The model is trained using Cross-Entropy Loss and the Adam optimizer.

## 🇲🇽 Español

Para la implementación práctica utilizamos Python y PyTorch para construir y entrenar modelos recurrentes para clasificación de secuencias.

Utilizamos el conjunto de datos de dígitos escritos a mano de scikit-learn.

Cada imagen tiene un tamaño de 8 por 8 píxeles.

Para este experimento, interpretamos intencionalmente cada imagen como una secuencia de ocho filas, con ocho valores de píxel como características en cada paso temporal.

La RNN y la LSTM procesan esos ocho pasos secuencialmente.

El estado oculto final se utiliza como un resumen de toda la secuencia y se pasa a una capa lineal de clasificación.

El modelo se entrena utilizando Cross-Entropy Loss y el optimizador Adam.

---

# SLIDE 10 — CONCLUSION & LIMITATIONS

## 🇺🇸 English

To conclude, RNNs are useful architectures for processing sequential data because they maintain information from previous time steps.

However, basic RNNs can struggle with long-term dependencies.

LSTMs resolve this limitation by introducing a cell state and gates that control what information should be forgotten, stored, and exposed as output.

This makes them useful for applications involving language, speech, time series, and other sequential data.

However, LSTMs also introduce greater computational complexity, so the architecture should be selected according to the characteristics of the problem.

## 🇲🇽 Español

Para concluir, las RNN son arquitecturas útiles para procesar datos secuenciales porque mantienen información de pasos anteriores.

Sin embargo, las RNN básicas pueden tener dificultades con dependencias de largo plazo.

Las LSTM abordan esta limitación mediante un estado de celda y diferentes puertas que controlan qué información olvidar, almacenar y exponer como salida.

Esto las hace útiles para aplicaciones relacionadas con lenguaje, voz, series temporales y otros datos secuenciales.

Sin embargo, las LSTM también introducen una mayor complejidad computacional, por lo que la arquitectura debe seleccionarse de acuerdo con las características del problema.

---

# SLIDE 11 — REFERENCES

## 🇺🇸 English

These are the main references we used for the theoretical concepts and architectures presented today.

## 🇲🇽 Español

Estas son las principales referencias que utilizamos para los conceptos teóricos y las arquitecturas presentadas.

---

# SLIDE 12 — THANK YOU

## 🇺🇸 English

Thank you for your attention.

Are there any questions?

## 🇲🇽 Español

Gracias por su atención.

¿Tienen alguna pregunta?


---
---
---
---
---
---
---
---
---
---
---
---
---
---



## 2. Ingeniería de Datos / Data Engineering

**[ES]** Un dígito de $8\times8$ píxeles se descompone en **$T=8$ pasos de tiempo** (filas horizontales) y **$F=8$ características**. Los píxeles se normalizan a $[0, 1]$ dividiendo entre 16. El dataset se divide en 70% entrenamiento, 15% validación y 15% pruebas de forma estratificada.

**[EN]** An $8\times8$ pixel digit is broken down into **$T=8$ time steps** (horizontal rows) and **$F=8$ features**. Pixels are normalized to $[0, 1]$ by dividing by 16. The dataset is split into 70% training, 15% validation, and 15% testing with stratification.

---

## 3. Arquitectura de Celdas: RNN vs. LSTM / Cell Architecture

**[ES]** 
* **RNN Simple:** Actualiza su memoria con `tanh`. Sufre de desvanecimiento de gradiente en secuencias largas.
* **LSTM:** Usa un estado de celda y compuertas (olvido, entrada, salida) para retener información a largo plazo de forma inteligente.

**[EN]** 
* **Simple RNN:** Updates memory using `tanh`. Suffers from vanishing gradients on long sequences.
* **LSTM:** Uses a cell state and gates (forget, input, output) to smartly retain long-term information.

---

## 🧠 Modelo Secuencial / Sequence-to-Class Model

**[ES]** La red lee las 8 filas paso a paso. Al llegar a la última fila, el estado oculto final ($h_T$) actúa como un vector resumen de todo el dibujo. Una capa lineal convierte este resumen en 10 logits (dígitos del 0 al 9).

**[EN]** The network reads the 8 rows step by step. Upon reaching the final row, the final hidden state ($h_T$) acts as a summary vector of the entire drawing. A linear layer turns this summary into 10 logits (digits 0 to 9).

---

## 4. Entrenamiento / Training

**[ES]** Entrenado con optimizador **Adam** y **BPTT (Backpropagation Through Time)** para propagar el error a través de los 8 pasos temporales. Se usa un tamaño oculto de $H=64$ para evitar sobreajuste.

**[EN]** Trained using the **Adam** optimizer and **BPTT (Backpropagation Through Time)** to propagate the error across the 8 time steps. A hidden size of $H=64$ is used to prevent overfitting.

---

## 5. Resultados y Conclusiones / Results and Conclusions

**[ES]** Ambos modelos superan el **96%-97% de precisión (Accuracy)**. La RNN simple destaca aquí por tener menos parámetros (~5.3k vs ~19.3k) y entrenar más rápido, ya que para secuencias cortas ($T=8$) la complejidad extra de la LSTM no es estrictamente necesaria.

**[EN]** Both models achieve over **96%-97% accuracy**. The simple RNN shines here with fewer parameters (~5.3k vs ~19.3k) and faster training, because for short sequences ($T=8$), the extra complexity of the LSTM is not strictly required.



