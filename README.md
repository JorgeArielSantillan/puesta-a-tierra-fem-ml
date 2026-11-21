# ⚡ Puesta a tierra en suelo de dos capas: elementos finitos + machine learning

**Autor:** Jorge Ariel Santillán

![La jabalina se clava y la resistencia cae al llegar a la capa húmeda](docs/jabalina_clavandose.gif)

## El problema

Una jabalina de puesta a tierra tiene que tener una resistencia baja para que, ante una falla, las tensiones sobre las personas sean seguras. La fórmula clásica (Dwight) supone un suelo uniforme, pero el suelo real tiene capas: por ejemplo, tierra seca arriba y una napa húmeda más abajo. En esos casos la fórmula puede equivocarse por un factor de 10 o más, y las aproximaciones de ingeniería (resistividad equivalente) se desvían hasta un 44 %.

## La solución

1. **Simulación por elementos finitos** del problema axisimétrico con herramientas libres: geometría y malla paramétricas en **Gmsh** (punta semiesférica, refinamiento en la jabalina, la punta y la interfaz) y solución en **Elmer FEM**.
2. **Validación** contra un motor analítico propio (método de los momentos con la función de Green de dos capas de Sunde): diferencia menor al 0,5 %.
3. **Estudio de convergencia de malla por energía:** la potencia disipada P = V²/R se calcula para mallas cada vez más finas; con 2 mm en las zonas críticas el error estimado es 0,08 % y el tiempo por caso no aumenta.
4. **Barrido automático de 600 casos** (Python controla Gmsh + Elmer, en paralelo, con reanudación y control de errores).
5. **Surrogate:** la fórmula de ingeniería más una corrección aprendida por un ensamble de redes neuronales. Predice R en milisegundos en lugar de segundos por simulación.

![Resistencia en función de la profundidad y campo en tres etapas](docs/resumen_vitrina.png)

## Resultados

| Método | Error medio en R | Peor caso |
|---|---|---|
| Dwight con suelo uniforme | > 100 % | > 1000 % |
| Dwight con resistividad equivalente | 9,9 % | 44 % |
| **Fórmula + corrección con ensamble de redes** | **0,48 %** | **3,3 %** |

*Conjunto de test: 120 simulaciones de Elmer que el modelo no vio durante el entrenamiento (80 % / 20 % de 600 casos).*

### Validación con 10 jabalinas nuevas

Para probar el modelo fuera del test, se simularon en Gmsh + Elmer 10 jabalinas con geometrías y suelos realistas: diámetros comerciales de 1/2" a 3/4", largos de 1,5 a 6 m y ρ₁ entre 30 y 1000 Ω·m. El modelo se entrenó solo con a = 8 mm y ρ₁ = 100 Ω·m, así que estos casos prueban que las variables adimensionales generalizan.

| | Error medio | Peor caso |
|---|---|---|
| Dwight con resistividad equivalente | 12,3 % | 37,8 % |
| Surrogate (500 casos) | 1,10 % | 7,5 % |
| **Surrogate (600 casos)** | **0,82 %** | **3,3 %** |
| Método de los momentos (control de Elmer) | 0,33 % | 0,41 % |

**Aprendizaje activo:** la primera validación encontró el punto débil del modelo. Una jabalina de 3 m con la napa a 2,9 m (solo los últimos 10 cm en la capa conductora) daba 7,5 % de error: ahí la corriente se concentra en la punta y R cae bruscamente, y el dataset tenía pocos casos con h/L ≈ 1. Se agregaron 100 simulaciones con h/L entre 0,85 y 1,1, se reentrenó y el error bajó a 3,3 %, sin cambiar nada más.

![Validación con 10 jabalinas nuevas](docs/validacion_10_jabalinas.png)

![Estudio de convergencia de malla por energía](docs/convergencia_elmer.png)

## Decisiones técnicas

- **Análisis dimensional:** R·L/ρ₁ depende solo de L/a, h/L y ρ₂/ρ₁. El modelo aprende en esas variables y generaliza a cualquier escala.
- **Física + datos:** la red no aprende R desde cero, sino la corrección sobre la fórmula de ingeniería. Así el error medio baja de 9,9 % a 0,48 %.
- **Punta semiesférica:** con punta plana, la densidad de corriente en la punta crece sin límite al refinar la malla (singularidad de esquina); con punta curva converge.

  ![Punta plana contra punta semiesférica](docs/punta_plana_vs_curva.png)
- **Dominio finito:** el potencial cero a distancia finita se corrige con ρ₂/(2π·R_dominio).
- **Muestreo dirigido:** se agregaron casos donde la punta queda cerca de la interfaz, la zona donde R cambia más bruscamente; la validación externa mostró dónde hacían falta más (ver arriba).

## Tecnologías

Gmsh · Elmer FEM · Python (NumPy, SciPy, scikit-learn, Matplotlib) · PowerShell

## Código fuente

El código es privado y está **disponible a pedido** para procesos de selección.

Contacto: georgesantillan1617@gmail.com · https://www.linkedin.com/in/jorge-ariel-santill%C3%A1n-235593183/
