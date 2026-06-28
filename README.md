# Proyecto final: Matching bipartito 4x4 del sector Minería de México como QUBO resuelto con QAOA

Qubit.mx, QMexico Summer School 2026.

Este repositorio formula un problema real del sector Minería de México como un problema de matching bipartito 4x4, lo expresa como QUBO, lo valida con un método clásico exacto y lo resuelve con QAOA en un simulador local.

## Contenido del repositorio

```text
.
├── proyecto_final_mineria_4x4.ipynb   notebook principal
├── data/
│   └── dataset_real_4x4.csv           dataset en formato largo
├── requirements.txt                   dependencias de Python
├── .gitignore                         archivos ignorados por git
├── LICENSE                            licencia MIT
└── README.md                          este archivo
```

## 1. El dataset

### Fuente

Los datos provienen del portal DataMéxico de la Secretaría de Economía del Gobierno de México, en el perfil del sector Minería (Mining, quarrying, and oil and gas extraction):

https://www.economia.gob.mx/datamexico/es/profile/industry/mining-quarrying-and-oil-and-gas-extraction

Fecha de consulta: junio de 2026.

Licencia: datos públicos abiertos del Gobierno de México. No contienen información personal identificable.

### Por qué es apto para QUBO

El dataset tiene dos lados naturales: los años recientes del sector (2021-2024) y los tamaños de empresa. Seleccionamos cuatro de cada uno para mantener la instancia en 4x4. El score se construye con variables observables disponibles en el portal (PIB, empleo, salario, permanencia, IED), y la decisión binaria x_ij = 1 modela el emparejamiento uno a uno. Los datos son públicos, agregados y no contienen información personal.

## 2. Formulación del problema

### Conjuntos

A = años recientes del sector Minería: 2021, 2022, 2023, 2024.

B = tamaños de empresa: Micro (hasta 10 personas), Pequeña (11 a 50), Mediana (51 a 250), Grande (251 y más).

### Variable de decisión

x_ij = 1 si el año i se empareja con el tamaño de empresa j, y 0 en otro caso.

### Score

El score de cada par se calcula con la fórmula:

S_ij = 0.30 (PIB) + 0.25 (Empleo) + 0.20 (Salario) + 0.15 (Empresas) + 0.10 (Exportaciones)

Componentes:

PIB: producto interno bruto anual del sector, depende del año i.

Empleo: fuerza laboral promedio del sector en el año i.

Salario: salario mensual promedio del sector en el año i.

Empresas: porcentaje de permanencia del personal según el tamaño de empresa j.

Exportaciones: inversión extranjera directa (IED) del sector en 2024. Es un valor global, así que entra como factor constante normalizado a 1.0.

Cada componente se normaliza al rango [0, 1] antes de combinarse, para que los pesos sean comparables.

### Por qué es un matching bipartito

Queremos emparejar cada elemento de A con exactamente un elemento de B y viceversa, maximizando la suma de scores. Eso es un matching bipartito perfecto, equivalente al problema de asignación.

### Por qué se formula como QUBO

El objetivo es lineal en las variables binarias y las restricciones son igualdades de suma. Las igualdades se convierten en penalizaciones cuadráticas:

E(x) = - suma de S_ij x_ij + lamA suma_i (suma_j x_ij - 1)^2 + lamB suma_j (suma_i x_ij - 1)^2

El resultado es una función cuadrática de variables binarias, es decir, un QUBO. Con lamA = lamB = 5.0 (mayor que el rango del score, que está en [0, 1]) el mínimo del QUBO coincide con una asignación factible de score máximo.

## 3. Interpretación de los resultados

Mejor asignación encontrada (clásico exacto y QAOA local coinciden):

```text
2021 -> Grande
2022 -> Mediana
2023 -> Micro
2024 -> Pequeña
```

Score total de la mejor asignación: 2.5775. Energía mínima del QUBO: -2.5775.

La asignación cumple todas las restricciones: cada año se empareja con un solo tamaño y cada tamaño con un solo año.

QAOA local sí observa el óptimo clásico entre sus mediciones, aunque con baja probabilidad porque la profundidad usada es p = 1. En una instancia de 16 variables esto es esperable.


## 4. Limitaciones y advertencia ética

El modelo combina variables de naturaleza distinta mediante normalización simple y pesos fijos, lo cual es una simplificación fuerte. La componente de exportaciones entra como constante porque solo existe un valor global del sector, así que en la práctica no diferencia entre pares.

Los resultados de QAOA no deben presentarse como una recomendación real de política pública, inversión o asignación de recursos. Son un ejercicio didáctico para practicar el pipeline QUBO, validación clásica y QAOA local.

## 5. Cómo ejecutar

Clona el repositorio y abre `proyecto_final_mineria_4x4.ipynb` en Google Colab o en Jupyter local. Ejecuta las celdas en orden. La celda de instalacion cubre todas las dependencias. No se necesita cuenta de IBM Quantum.
