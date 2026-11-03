Integrantes: Santiago Devesa
IA declarada: Gemini, DeepSeek
Pista: Reglas
Publicación en LinkedIn: https://lnkd.in/p/dP6cEVJX


1. Formulación y PEAS
La pregunta. Para cada caso, ¿el jugador duplicará su valor de mercado en dos años? SÍ significa que lo duplica, NO significa que no.

PEAS	Su sistema
Medida de rendimiento	La etiqueta real del dataset (1 = SÍ duplica, 0 = NO duplica). Evaluamos si el sistema predice correctamente el salto de valor.
Entorno	La secretaría técnica del club.
Actuadores	El sistema imprime la decisión final: 1 "SÍ (Duplica valor)" o 0 "NO (No duplica)".
Sensores	Recibe 3 datos clave: Edad, Minutos jugados y Goles + asistencias cada 90 mins.
Objetivo y métrica.

Objetivo: Que el jugador realmente explote y duplique su valor.

Métrica: La etiqueta "Resultado real" (1 = SÍ, 0 = NO) que figura en el dataset.

¿Son lo mismo? No. La métrica es una foto del pasado y el objetivo es predecir el futuro. Usamos la métrica como un proxy (representante) del objetivo.

2. El sistema
Cómo decide. Usamos un sistema de reglas Si-entonces:

R1 (Talento Joven): SI Edad <= 20 Y Minutos > 1000 Y G+A >= 0.55 ENTONCES SÍ.

R2 (Rendimiento): SI Edad <= 23 Y Minutos > 2000 Y G+A > 0.6 ENTONCES SÍ.

R3 (Descarte por Edad): SI Edad > 25 ENTONCES NO. (La sacamos analizando solo el conjunto de entrenamiento: los mayores de 25 nunca duplicaron su valor).

R4 (Descarte por Volumen): SI Minutos < 1000 ENTONCES NO.

Por defecto: Si no cumple R1 ni R2, o cae en R3/R4, la respuesta es NO.

Variables que usa y por qué. Usa edad para filtrar proyección, minutos para la confianza del cuerpo técnico, y G+A para medir cómo juega genuinamente el jugador. No usamos "Resultado real" porque es la respuesta.

Cómo se corre en vivo.

Recibimos los 3 datos del caso nuevo.

Evaluamos las reglas en orden estricto (R3 y R4 primero para descartar rápido, luego R1 y R2).

Damos la respuesta.

Dos casos de ejemplo.

Caso inventado	Valores	Salida del sistema
A	19 años, 1500 min, 0.8	SÍ (Cumple R1)
B	27 años, 2000 min, 0.9	NO (Cumple R3)
3. Medición
Entrenamiento (casos 1–14)

Predice SÍ	Predice NO
Real SÍ	VP = 2	FN = 0
Real NO	FP = 2	VN = 10
Prueba (casos 15–20)

Predice SÍ	Predice NO
Real SÍ	VP = 1	FN = 0
Real NO	FP = 0	VN = 5
Métrica	Entrenamiento	Prueba
Exactitud	12/14 = 85.7%	6/6 = 100%
Precisión	2/(2+2) = 50%	1/(1+0) = 100%
Sensibilidad	2/(2+0) = 100%	1/(1+0) = 100%
Especificidad	10/(10+2) = 83.3%	5/(5+0) = 100%
Entrenamiento contra prueba. Hay una gran diferencia: en entrenamiento la exactitud es del 85.7% y en prueba es del 100%. Esto indica que las reglas generalizan bien (no hay sobreajuste), porque funcionan mejor con datos que el sistema nunca vio. (Aclaración: el caso 19 es de prueba y lo usamos solo para verificar que la R3 funcionaba, no para diseñarla; si no, sería sobreajuste).

4. El error más caro
El error más caro es el Falso Positivo (FP): decirle al club que un jugador va a duplicar su valor cuando en realidad no lo hará. No solo es un error teórico, sino que tiene consecuencias directas: el club invertirá tiempo, recursos y un cupo de extranjero en un jugador que no rendirá, perdiendo la oportunidad de hacerlo con otro. Para evitar esto, implementamos la R3 (Descarte por Edad > 25). El caso 19 (26 años, 0.92 G+A) es el ejemplo perfecto: si no tuviéramos esta regla, el sistema lo clasificaría como SÍ (por su alto rendimiento), pero la realidad dice que NO. Preferimos un Falso Negativo a un Falso Positivo.

5. ¿Comparado contra qué?
Línea base trivial: Si el sistema dijera siempre "NO" (porque la mayoría de las etiquetas son 0), la exactitud en prueba sería de 5/6 = 83.3%. Nuestro sistema tiene 100% de exactitud en prueba, por lo que supera la línea base.

Checklist (Clase 6)	Respuesta
¿Qué datos?	20 casos históricos de juveniles (14 train / 6 test).
¿Qué métrica?	Exactitud, Precisión, Sensibilidad y Especificidad.
¿Comparado contra qué?	Contra la línea base de "decir siempre NO" (83.3%).
¿Quién lo midió?	El grupo (Santiago Devesa), usando el dataset de la cátedra.
6. Riesgo: el camino completo
Datos → Modelo → Decisión → Consecuencia

Datos: 20 casos de juveniles. Riesgo: Si el club solo nos da datos de jugadores de cierta posición (ej. delanteros), el modelo no aprenderá sobre defensores.

Modelo: Nuestras reglas. Riesgo: Si el club cambia su política (ej. ahora apuesta a veteranos), las reglas quedan obsoletas.

Decisión: Contratar o no.

Consecuencia: Un jugador que duplica su valor (éxito) o un fracaso deportivo/económico.

Riesgo concreto: El sesgo de datos. Si en los 20 casos no hay jugadores de 27 años que sí duplicaron su valor, nuestro modelo nunca aprenderá a detectarlos.

7. Bitácora
Uso de la IA

#	Qué le pedimos	Qué aceptamos	Qué rechazamos y por qué
1	Analizar los datos y sugerir reglas	Usar la edad como filtro principal.	Nos sugirió una neurona con pesos, pero la rechazamos porque el caso 19 (26 años, 0.92) la rompía.
2	Armar la matriz de confusión	La lógica de los cálculos.	Nos sugirió usar "Exactitud" como métrica principal, pero la rechazamos porque el Hackatón pide enfocarse en el error más caro (FP).
Corridas contra el evaluador

Corrida	Nota	Agujero que encontró	Qué cambiamos
1	7	El caso 19 es de prueba y lo usaron para diseñar R3. La matriz de entrenamiento no coincide con las reglas.	Corregimos la matriz de entrenamiento (VP=2, FP=2) y justificamos que R3 se diseñó mirando solo entrenamiento. Agregamos la línea base trivial.
2	7	La matriz de prueba tiene los números de la de entrenamiento.	Corregimos la matriz de prueba: 6 casos, VP=1, FN=0, FP=0, VN=5.
