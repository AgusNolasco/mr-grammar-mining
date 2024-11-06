# Mineria de Gramaticas para la generacion de Relaciones Metamorficas

## Resumen
La aplicacion del testing metamorfico esta inheremente ligado a la uso de relaciones metamorficas. Las mismas son propiedades que capturan una relacion entre dos, o mas, ejecuciones del software. Por ejemplo, en el contexto de programas orientados a objetos, para una implementacion de pilas la relacion metamorfica que dice que realizar pop() despues de realizar push(Object) dejara a la pila en el estado previo a realizar push. Por otro lado, para implementacion de funciones matematicas, nos interesan propiedades como sin(x) = -sin(-x), o sum(x,y) = sum(y,x). Estos tipos de relaciones metamoricas mostrados corresponden a dominios distintos, para la primera el dominio son secuencias de metodos equivalentes donde se evalua el estado final del objeto y, opcionalmente, el resultado obtenido del mismo. Mientras que para las otras propiedades se espera evuluar que los resultados de las operaciones sean iguales luego de una pertubacion en sus argumentos.

Por ejemplo, para la clase:

```
public class BoundedStack {

	private final static int DEFAULT_SIZE = 10;

	private final Object[] elements = new Object[DEFAULT_SIZE];

	private int index = -1;

	public BoundedStack() { }

	public void push(Object object) {
		if (isFull()) {
			throw new IllegalStateException();
		}
		elements[++index] = object;
	}

	public Object pop() {
		if (isEmpty()) {
			throw new IllegalStateException();
		}
		Object ret_val = elements[index];
		elements[index] = null;
		index--;
		return ret_val;
	}
	
	public boolean isFull() {
		return index == elements.length - 1;
	}
	
	public boolean isEmpty() {
		return index == -1;
	}
}
```

Esperariamos una gramatica con la siguente pinta:

$$
\begin{flalign}
    \texttt{MR} &::= \texttt{OBJ\\_EQ}~|~\texttt{RET\\_EQ} \\
    \texttt{OBJ\\_EQ} &::=\texttt{mod\\_method\\_seq} \texttt{ REL\\_OBJ } \texttt{mod\\_method\\_seq} \\
    \texttt{RET\\_EQ} &::=\texttt{mod\\_method\\_seq; pure\\_method} \texttt{ REL\\_RET } \texttt{mod\\_method\\_seq; pure\\_method} \\
    \texttt{REL\\_OBJ} &::= \texttt{\`=='}\\
    \texttt{REL\\_RET} &::= \texttt{\`=='}\\
    \texttt{mod\\_method\\_seq} &::= \texttt{mod\\_method; mod\\_method\\_seq}~|~\epsilon \\
    \texttt{mod\\_method} &::= \texttt{push(OBJ)}~|~\texttt{pop()} \\
    \texttt{pure\\_method} &::=\texttt{isFull()}~|~\texttt{isEmpty()} \\
    \texttt{OBJ} &::= \texttt{pop()}~|~\texttt{Object}
\end{flalign}
$$

## Hipotesis
Se espera que un modelo de lenguaje orientado de codigo, por ejemplo CodeBERT, sea capaz de generar una gramatica que capture el lenguaje de las relaciones metamorficas relevantes para una clase dada.

## Objectivos preeliminares
En el caso de que no sea posible obtener directamente la gramatica a partir del modelo de lenguaje, se podria opcionalmente, obtener informacion sobre las operaciones de la clase bajo analisis. Esta informacion puede ser los tipos de sus argumentos, el tipo de retorno, si la operacion produce cambios sobre el objeto o si es un funcion pura (sin efectos colaterales). Luego, con esta informacion uno podria crear de forma automatica la gramatica, o en su defecto, proveer a otro modelo con dicha informacion para generar la gramatica.

## Técnicas a aplicar
Actualmente, la tecnica que se considero más viable es la utilización de *prompts*. La idea es diseñar uno o varios prompts con el objetivo de resolver este problema en uno o varios pasos. Extraccion de informacion relevante sobre las operaciones y generacion de una gramatica de propiedades metamorficas. Otras tecnica que podria utilizarse el fine-tuning sobre algun modelo existente, la idea seria apuntar a una tarea de *text summarization*, enfocando el resumen a obtener caracteristicas sobre las operaciones de una clase, para posteriormente construir la gramatica. El principal problema de esta tecnica es la necesidad de *training data*. Otra tecnica, es la utilización de few-shot, debido a la poca cantidad de ejemplos con los que se cuenta, para realizar fine-tuning sobre un modelo existente.

## Planificación

### Semana 1-2
Para las primeras semanas se busca hacer algunos experimentos exploratorios para evaluar la viabilidad del uso de LLMs para la generación de gramáticas que capturen MRs. Para ello utilizaremos ChatGPT con un *prompt* que detalle como debe ser la construcción de la gramática. El prompt contara con instrucciones para distinguir clases *stateful* y *stateless*, de forma que clases *stateful* contaran con MRs que modifiquen el estado concreto de un objeto, mientras que las *stateless* no. También se le proporcionara información de que debe hacer con los parametros de las operaciones. Por ejemplo la operacio `push(Object)` de `BoundedStack` toma un `Object`, el cual se podra obtener de manera aleatoria o mediante otros mecanismos de la clase, como por ejemplo una llamada al metodo `pop()`, el cual devuelve un `Object`. Para alguna clase que implemente funciones aritmeticas, es seguro que la constante $\pi$ estara definida como un atributo, entonces si tenemos `sin(float)`, esperariamos que ese `float` se obtenga de manera aleatoria, o tomando el atributo de la clase $\pi$ quedando `sin`($\pi$), tambien podria obtenerse a traves de una invocación al metodo `cos(float)`. Para tipos como, por ejemplo, `Integer` se espera poder realizar pertubaciones, extendiendo la gramática con operaciones aritmeticas entre `Integers`, y asi con cualquier otro tipo para el cual se conozcan operaciones.

### Semana 3-4
En las siguentes semanas la idea seria utilizar los ejemplos generados y refinados a partir del uso de prompts en ChatGPT para realizar few-shot sobre algun modelo de lenguaje con el objeto de entrenarlo en la tarea de generar gramaticas que capturen las posibles propiedades metamorficas de una clase dada.
En la carpeta 'experiments' se encuentran distintas clases Java y sus respectivas gramaticas obtenidas a partir del uso de ChatGPT. Para ello, como se menciono anteriormente se utilizo un prompt. Si bien en varios casos complejos ChatGPT no supo darnos una gramatica que cumpliera con los solicitado en el prompt, mediante algunos comandos adicionales se pudo obtener un gramatica mas cercana a lo esperado. Estos seran los ejemplos utilizados para realizar el few-shot.

# Otras propuestas

## Deteccion de precondiciones para propiedades metamorficas

**Resumen**: en trabajos anteriores se encontro que ciertas propiedades metamorficas eran validas en determinados escenarios, pero en otros no. Lo que se busca es usar un modelo de lenguaje que tome como entrada la propiedades bajo analisis y los escenarios donde se probo la propiedad, y que nos de como salida una formula logica que distinga dicho escenarios. Por ejemplo: la propiedad `pop();push(Object) = \empty_seq` solo es valida si el elemento que se inserta en la pila es el mismo que se quito, es decir `x = pop() -> push(x) = \empty_seq`.

## Generacion automatica de aserciones

**Resumen**: se plantea realizar procesamiento de lenguaje natural sobre los comentarios de alguna operacion para, dado un input para la operacion, obtener una asercion valida para ese caso. Por ejemplo para:
```
/**
* @return the maximum of two integers
*/
public int max(int x, int y) {
  return x > y ? x : y;
}
```

y los inputs `x=3` y `y=10`, se espera obtener una asercion como `assertEquals(max(x,y), y)`
