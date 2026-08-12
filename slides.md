---
theme: seriph
background: https://images.unsplash.com/photo-1517241034903-9a4c3ab12f00?q=80&w=1740&auto=format&fit=crop&ixlib=rb-4.1.0&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
class: text-center
highlighter: shiki
lineNumbers: false
drawings:
  persist: false
css: unocss
---

# Patrones de diseño
---

# Definición

> “Cada patrón describe un problema que se repite una y otra vez en nuestro ambiente,
> y describe la solución para ese problema, de forma que esta solución se pueda utilizar
> un millón de veces, sin utilizarla dos veces de la misma forma.”

<br>

## ¿Qué son los patrones de diseño?

Un **patrón (Pattern)** describe una **solución a un problema** que surge al desarrollar software dentro de un contexto particular.

### En otras palabras

- Son **descripciones de soluciones** a problemas de diseño general que se repiten una y otra vez.
- Son **decisiones de diseño** que le sirvieron a otros diseñadores.
- Generan un **nuevo vocabulario de diseño**.

---

## Respuestas que brinda un patrón

- **Problema**  
  ¿Qué problema de diseño soluciona?

- **Solución**  
  ¿Cómo solucionar el problema?

- **Contexto**  
  Precondiciones para que se pueda aplicar el patrón.

- **Restricciones**  
  ¿Qué afecta la aplicación del patrón?

---

## ¿Por qué es importante conocer los patrones?

- ♻️ **Reutilizar conocimiento**  
  Aprovechar soluciones que ya demostraron ser útiles.

- 💬 **Establecer un lenguaje común**  
  Facilitar la comunicación con el equipo al discutir diseños e implementaciones.

- 🧩 **Entender frameworks y librerías**  
  Los patrones de diseño se encuentran presentes en todo tipo de frameworks y librerías.

---
layout: image-right
image: ./imgs/libroPatterns.jpg
backgroundSize: 75% 75%
---

# Clasificación de patrones — GoF
## 🔄 Comportamiento
+ Interacción entre objetos

## 🏗️ Creacionales
 + Creación de objetos

## 🧩 Estructurales
+ Composición de clases y objetos

---
layout: default
---

# Clasificación de patrones

<div class="grid grid-cols-3 gap-8">


<div>

##  Comportamiento

- Strategy
- Template Method
</div>

<div>

##  Creacionales

- Singleton
- Factory Method
- Builder

</div>

<div>

##  Estructurales

- Facade

</div>
</div>
---
---

# Strategy

+ Buscamos definir una familia de algoritmos, encapsular cada uno, y hacer que sean intercambiables​.

+ Esta estrategia permite variar al algoritmo en forma independiente de quien lo utiliza.

---

# ¿Cuándo aplica?​

+ Muchas clases relacionadas difieren en su comportamiento. Strategy provee una forma de configurar una clase con uno de muchos comportamientos​

+ Es necesario utilizar variantes de un algoritmo​

+ Un algoritmo utiliza datos que la clase cliente no necesita conocer. Strategy protege a esa clase de datos y estructuras complejas​

+ Una clase define múltiples comportamientos dependientes de sentencias condicionales. En lugar de utilizar estas pasarlas a una Strategy​

​

---

# Ejemplo de codigo donde utilizaremos Strategy

```csharp {all|1|4|all}
Compra c = new() { Total = 100, Pais = "uy" };​
Console.WriteLine($"Impuestos en UY: {c.CalcularImpuestos()}");​
​
c.Pais = "us";​
Console.WriteLine($"Impuestos en USA: {c.CalcularImpuestos()}");​​
```

```csharp {all|8,10|12,14|all}
public class Compra​
    {​
        public string Pais { get; set; }​

        public int CalcularImpuestos()​
        {​
            var impuestos = 0;​
            if (Pais == "uy")​
            {​
                // Cada una de estas, es una implementacion del cálculo​
            }​
            else if (Pais == "us")​
            {​
	              // Cada una de estas, es una implementacion del cálculo​
            }​
            return impuestos;​
        }​
}​
```
---
