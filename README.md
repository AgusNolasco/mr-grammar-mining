# Mineria de Gramaticas para la generacion de Relaciones Metamorficas

**Resumen**: la aplicacion del testing metamorfico esta inheremente ligado a la uso de relaciones metamorficas. Las mismas son propiedades que capturan una relacion entre dos, o mas, ejecuciones del software. Por ejemplo, en el contexto de programas orientados a objetos, para una implementacion de pilas la relacion metamorfica que dice que realizar pop() despues de realizar push(Object) dejara a la pila en el estado previo a realizar push. Por otro lado, para implementacion de funciones matematicas, nos interesan propiedades como sin(x) = -sin(-x), o sum(x,y) = sum(y,x). Estos tipos de relaciones metamoricas mostrados corresponden a dominios distintos, para la primera el dominio son secuencias de metodos equivalentes donde se evalua el estado final del objeto y, opcionalmente, el resultado obtenido del mismo. Mientras que para las otras propiedades se espera evuluar que los resultados de las operaciones sean iguales luego de una pertubacion en sus argumentos.

**Hipotesis**: se espera que un modelo de lenguaje orientado de codigo, por ejemplo CodeBERT, sea capaz de generar una gramatica que capture el lenguaje de las relaciones metamorficas relevantes para una clase dada.

**Objectivos preeliminares**: en el caso de que no sea posible obtener directamente la gramatica a partir del modelo de lenguaje, se podria opcionalmente, obtener informacion sobre las operaciones de la clase bajo analisis. Esta informacion puede ser los tipos de sus argumentos, el tipo de retorno, si la operacion produce cambios sobre el objeto o si es un funcion pura (sin efectos colaterales). Luego, con esta informacion uno podria crear de forma automatica la gramatica, o en su defecto, proveer a otro modelo con dicha informacion para generar la gramatica.

**Tecnicas a aplicar**: actualmente, la tecnica que considero mas viable es la de *prompt engineering*. La idea es diseñar uno o varios prompts con el objetivo de resolver este problema en varios pasos. Extraccion de informacion relevante sobre las operaciones y generacion de una gramatica de propiedades metamorficas. Otras tecnica que podria utilizarse el fine-tuning sobre algun modelo existente, la idea seria apuntar a una tarea de *text summarization*, enfocando el resumen a obtener caracteristicas sobre las operaciones de una clase, para posteriormente construir la gramatica. El principal problema de esta tecnica es la necesidad de *training data*. Otra tecnica, es la utilizacion de few-shot, debido a la poca cantidad de ejemplos con los que se cuenta, para realizar fine-tuning sobre un modelo existente.

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


$$
\begin{align}
    \texttt{MR} &::= \texttt{OBJ\\_EQ}~|~\texttt{RET\\_EQ} \\
    \texttt{OBJ\\_EQ} &::=\texttt{mod\\_method\\_seq} =\_{\texttt{obj}} \texttt{mod\\_method\\_seq} \\
    \texttt{RET\\_EQ} &::=\texttt{mod\\_method\\_seq; pure\\_method} =\_{\texttt{ret}} \texttt{mod\\_method\\_seq; pure\\_method} \\
    \texttt{mod\\_method\\_seq} &::= \texttt{mod\\_method; mod\\_method\\_seq}~|~\epsilon \\
    \texttt{mod\\_method} &::= \texttt{push(OBJ)}~|~\texttt{pop()} \\
    \texttt{pure\\_method} &::=\texttt{isFull()}~|~\texttt{isEmpty()} \\
    \texttt{OBJ} &::= \texttt{pop()}~|~\texttt{Object}
\end{align}
$$

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
