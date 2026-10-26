# 🧩 Propuesta de solución de Sudoku de Alejandro Sobrino

> Este notebook de Jupyter contiene mi propuesta de modificación al código del algorítmo genético para solución de Sudoku como parte del curso de Agentic A.I. de Actumlogos. 

**Autor:** Alejandro Sobrino (alexsob@gmail.com)
**Fecha:** 25 de Octubre de 2026

## Problemáticas encontradas en la versión original del código.

### Función fitness inadecuada

La función fitness original, propone evaluar los elementos únicos de cada fila y columna (calculando la transversal de la matriz original) convirtiendo el arreglo a un set (que descarta los elementos repetidos) y contando su longitud. Esta función está incompleta, ya que también es necesario evaluar la repetición de elementos por bloque.

#### Mejora propuesta

Mi solución de fitness tiene dos cambios importantes, primero cambia el enfoque de evaluación a contar los elementos repetidos (vs. contar únicos en la versión original); contar repetidos es igual de útil que contar únicos pero decidí hacer el cambio para practicar Python. Adicionalmente, se incluye una rutina para evaluar los bloques encontrando elementos repetidos. El resultado es el siguiente:

```
def fitness_repeated(grid):
    #versión Alejandro Sobrino
    '''
    Esta función de fitness busca los números repetidos en filas, columnas y cuadros.
    El objetivo es MIN(Fitness) == 0
    '''

    #Sacando cuenta de repetidos para columnas y filas
    row_repeated = sum(len(row)-len(set(row)) for row in grid)
    col_repeated = sum(len(col)-len(set(col)) for col in grid.T)
    
    #Calcula números unicos por cuadricula
    squares_repeated = 0
    for square_x in range(3):
        for square_y in range (3):
            #Check for repeats in the square
            square_uniques = []
            for x_i in range(square_x*3,(square_x+1)*3):
                for y_i in range(square_y*3,(square_y+1)*3):
                    if grid[y_i][x_i] not in square_uniques:
                        square_uniques.append(grid[y_i][x_i])
                    else:
                        squares_repeated = squares_repeated + 1
                        

    return row_repeated + col_repeated + squares_repeated

```

### Falta de mecanismo para evitar caer en máximos locales

Después de corregir la función de fitness, correr el código algunas decenas de veces sin encontrar una respuesta correcta y observar los sudokus resultantes, me di cuenta que el algoritmo original tiende a terminar en mínimos locales que dificilmente podrían mutar en el resultado correcto. Se intentaron diferentes estrategias sin éxito:

- Variar el mutation_rate despues de N generaciones sin cambios en el mejor fitness encontrado.
- Variar el tamaño de población despues de N generaciones sin cambios en el mejor fitness encontrado.
- Introducir poblaciones aleatorias para tratar de forzar un cambio significativo en la solución.

#### Mejora propuesta

Finalmente, implementé una estrategía que me permitió que las soluciones finales se acercaran más al fitness ideal: Introduje un mecanismo que guardaba "ancestros de progreso", es decir, cada best_population que acercaba al menos un paso al fitness ideal. Al completar X poblaciones sin progreso, el mecanismo intercambia la población actual por los ancestros que generaron alguna mejora, con la esperanza de que el algoritmo pueda recobrar una rama diferente del arbol de posibles cambios y soluciones. El mecanismo resultante:

```
if generations_stuck == 0:
            if len(ancestors) < (pop_size):
                ancestors.append(best_solution)
                #print(f"Saving progress ancestor: Current Pop fit avg {sum(fitness(x) for x in population)/len(population):5f} | Ancestors {sum(fitness(x) for x in ancestors)/len(ancestors):5f}, count({len(ancestors):3d})")

         
        if (generations_stuck>200):
            if(generations_stuck%5000 == 0):
                if(fitness(ancestors[-1]) < fitness(best_solution)):
                    ancestors.pop(0)
                    ancestors.append(best_solution)
                #print(f"Repopulating with ancestors: Current Pop fit avg {sum(fitness(x) for x in population)/len(population):5f} count:({len(population)}) | Ancestors {sum(fitness(x) for x in ancestors)/len(ancestors):5f}, count({len(ancestors):3d})")
                population = ancestors.copy()
                while len(population) < pop_size:
                    population = ancestors + population
                population = population[0:pop_size]
                random.shuffle(population)
```

Este mecanismo permitió llegar de resultados con un fitness mínimo promedio de 12.5 a mínimos de al rededor de 6; sin embargo nunca logró resolver el sudoku por completo, por lo que se implementó un mecanismo adicional. Calcular variaces las soluciones parciales a término y formar una población con decenas de mejores candidatos a término. Esto hizo posible encontrar la solución al sudoku. A continuación el código resultante:

```
# Calcular 100 mejores intentos con la configuración y luego convertirlo en una población
strong_population = []
start_time = time.perf_counter()
end_time = time.perf_counter()
for i in range(150):
    end_time = time.perf_counter()
    elapsed_time = end_time - start_time
    print(f"Elapsed Time: {elapsed_time:.6f}|Mejor intento número: {i:3d}")
    strong_population.append(genetic_algorithm(pop_size=200, max_generations=1500, mutation_rate=0.03))

print("OK, tengo las poblaciones")
genetic_algorithm(pop_size=400, max_generations=10000, mutation_rate=0.03,population=strong_population)
end_time = time.perf_counter()
print(end_time-start_time)
```

## Solución al Sudoku

Los mecanismos descritos anteiormente me permitieron encontrar la siguiente solución al sudoku:

```
[
    [5, 3, 4, 6, 7, 8, 9, 1, 2],
    [6, 7, 2, 1, 9, 5, 3, 4, 8],
    [1, 9, 8, 3, 4, 2, 5, 6, 7],
    [8, 5, 9, 7, 6, 1, 4, 2, 3],
    [4, 2, 6, 8, 5, 3, 7, 9, 1],
    [7, 1, 3, 9, 2, 4, 8, 5, 6],
    [9, 6, 1, 5, 3, 7, 2, 8, 4],
    [2, 8, 7, 4, 1, 9, 6, 3, 5],
    [3, 4, 5, 2, 8, 6, 1, 7, 9]
]
```

La solución propuesta tarda aproximadamente 1:07 horas en regresar la respuesta correcta. Experimentalmente tambien se comprobó que el algortimo propuesto no asegura el descubrimiento de la solución el 100% de las veces.