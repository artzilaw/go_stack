
Что из себя представляет REST API? Мы уже прошли много всего по HTTP и у нас может возникнуть несколько вопросов по ходу того, как мы будем проектировать наше приложение:
1)  Как называть патерн?
2) Какой HTTP-метод использовать?
3) Использовать тело запроса или query-параметр?
4) Что возвращать в теле запроса?
И прочие, вопросов будет возникать много, для этого как раз и существует RESP API, который и есть тот самый набор правил и принципов, который и должен дать ответы на все эти вопросы. Это не значит,что RESI API является единой парадигмой для построения приложения, это не значит,что он используется повсеместно, но какие-то части используются почти везде. 

После обсуждения всех принципов REST API мы это все закрепим на практике, вот конкретно,что у нас там будет:
![[rest_api_practice.png|540]]

Мы по сути будем брать код из проекта,который был ранее и оборачивать его в HTTP-сервер. 

Вот структура того,как мы будем это ве дело хранить:
![[rest_api_project_struct.png]]

Каждая задача будет из себя представлять структуру из следующего набора полей: Заголовок; Описание; Выполнена ли? ; Дата создания; Дата выполнения. 

Теперь поговорим про принципы  REST API(далеко не все,но основные):
0)  Клиент-серверная архитектура
    - Мы описываем наш сервер и предоставляем контракт клиентам для взаимодействия с нашим сервером
    - Контракт удовлетворяет требованиям REST API
	Что это значит? Мы должны соблюдать клиент-серверную архитектуру, и предоставить клиентам контракт. Что такое контракт? Контракт - набор http- патернов , набор методов,которые эти патерны принимают,наборы значений, которые хендлеры дополнительно требуют в качестве уточняющей информации, возвращаемые статус коды этих хендлеров, информация из хендлеров и тд. И этот самый контракт(не на сво) должен удовлетворять требованиям REST API
	 
![[client-server_rest_api.png|540]]

1)  Определенный стиль именования патернов(эндпоинтов):
	- Патерны именуются существительным
	- Существительное - название ресурса,над которым производится работа
	Вот у нас сейчас будет работа с ToDO-листом, у нас опреация производится над задачей,следовательно названием ресурса,над которым производится работа будет задача, следовально названия патерна будет "/tasks".  И получается в итоге,что этот патерн у нас будет одинаковым для всех хендлеров. То есть, никаких глаголов, строго существительное, над которым мы осуществляем работу. 
	
![[rest_api_name_endpoint.png|540]]

2)  Строго определенные HTTP-методы под каждый вид манипуляции над ресурсом. 
	-  Создание нового ресурса: POST
	-  Получение уже существующего ресурса: GET
	- Частичное изменение существующего ресурса: PATCH
	-  Полное изменение существующего ресурса: PUT
	-  Удаление существующего ресурса: DELETE
	Короче,нельзя выбирать рандомный метод,нужно выбирать метод под конкретную задачу. 
![[rest_api_methods.png|540]]

3) Строго определенные места для передачи дополнительной информации в запросе
	-  Создание нового ресурса: JSON в теле запроса
	-  Получение одного уже существующего ресурса: Индентификатор ресурса зашит в патерне 
	-  Получение всех существующих ресурсов: Дополнительная информация не нужна
	-  Получение всех существующих ресурсов,учитывая фильты: Query параметры
	-  Частичное изменение существующего ресурса: Идентификатор ресурса зашит в патерне + JSON в теле запроса
	-  Полное изменение существующего ресурса: Идентификатор ресурса зашит в патерне + JSON в теле запроса
	-  Удаление существующеего ресурса: Идентификатор ресурса зашит в патерне
	Что означает зашить в патерн?  Вот мы описали хендлер,который просто выводит некий `r.URL.Path`, так вот, это  метод нам возвращает некие значения после `/`,который мы указали в запросе после патерна,для наглядности вот код:
	
```go
	package main
	
	import (
		"fmt"
		"net/http"
	)
func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Println(r.URL.Path) // возвращаем в функцию как раз патерн и значения после него 
}

func main() {
	http.HandleFunc("/tasks/", handler) // "/" после патерна позволяет записывать после него еще что-то и запрос пройдет
	
	if err := http.ListenAndServe(":1488", nil); err != nil {
	fmt.Println(err)
	}
}
```

Мы можем потом все это по слешам распарсить и получать из этого самого патерна то,что нам надо. Как правильно исходя из наших правил выше оформить запросы в нашем пет проекте указано ниже:

![[rest_api_param_request.png|540]]

4) Строго определенная информация в HTTP теле ответа
	![[rest_api_fourth.png|664]]
	
	Каждая операция должна возвращать информацию в определенном формате, зачастую это JSON
	
5)  Строго определенная информация в HTTP статус-коде ответа
	![[rest_api_fiveth.png|664]]
	
	Каждой операции надо вернуть ее статус код, что чему конкретно указано на картинке выше. Перечислены не все статус коды,которые могут возникнуть.  И кстати,на ошибках указано сразу несколько вариантов, от того,что банально запрос от клиента неверный (400 Bad Request) или проблемы на серверной части(500 Internal Server Error), до спецефических, которые могут возникнуть от того,что такого элемента просто не,чтобы какое-то действие над ним совершить(404 Not Found)  или того,что элемент с таким идентификатором уже существует(409 Conflict). 

Раз правила обговорили,можно теперь писать код. Автор взял код с проекта с первого урока, сейчас оставлю тут то,что надо было тут импортировать(будет код автора,а не мой)

errors.go:
```go
package todo

var taskNotFound string = "задача не найдена"
```

list.go:
```go
package todo

type List struct {
	tasks map[string]Task
}

func NewList() *List {
	return &List{
		tasks: make(map[string]Task),
	}
}

func (l *List) AddTask(task Task) {
	l.tasks[task.Description] = task
}

func (l *List) ListTasks() map[string]Task {
	return l.tasks
}

func (l *List) DoneTask(description string) string {
	task, ok := l.tasks[description]
	if !ok {
		return taskNotFound
	}
	task.Done()
	l.tasks[description] = task
	return ""
}

  

func (l *List) DeleteTask(description string) string {
	_, ok := l.tasks[description]
	if !ok {
		return taskNotFound
	}
	delete(l.tasks, description)
	return ""
	}
```

task.go:
```go
package todo

import "time"

type Task struct {
	Description string
	Text string
	IsDone bool
	
	CreatedAt time.Time
	DoneAt *time.Time
}

func NewTask(description string, text string) Task {
	return Task{
		Description: description,
		Text: text,
		IsDone: false,
		CreatedAt: time.Now(),
		DoneAt: nil,
	}
}

func (t *Task) Done() {
	doneTime := time.Now()
	t.IsDone = true
	t.DoneAt = &doneTime
}
```

Как оно работает автор не объясняняет,но тут ничего сложного будто нет. 

Первым делом нам надо переписать файл с ошибками,так как на момент,когда код писался,мы не умели это делать

errors.go:
```go
package todo

import "errors"

var taskNotFound error = errors.New("task not found")
var taskAlreadyExists error = errors.New("task already exists")
```

Далее метод,где мы создаем новую таску проверяем на предмет того,есть ли уже элемент с заголовок,если да,то возвращаем ошибку

list.go:
```go
func (l *List) AddTask(task Task) error {
	if _, ok := l.tasks[task.Title]; ok {
		return ErrTaskAlreadyExists
	}
	l.tasks[task.Title] = task
	return nil
}
```

Еще надо переписать модуль, где у нас идет получение всех элементов мапы(чтобы не было доступа к исходной мапе,а только к копии)

list.go:
```go
func (l *List) ListTasks() map[string]Task {
	tmp := make(map[string]Task,len(l.tasks))
	for k, v := range l.tasks {
		tmp[k] = v
	}
	return tmp
}
```

Далее там еще осталось 2 метода(DeleteTask(),CompleteTask()), где мы должны переписать механизм возвращения ошибок

list.go:
```go
func (l *List) CompleteTask(title string) error {
	task, ok := l.tasks[title]
	if !ok {
		return ErrTaskNotFound
	}
	task.Complete()
	l.tasks[title] = task
	return nil
}

func (l *List) DeleteTask(title string) error {
	_, ok := l.tasks[title]
	if !ok {
		return ErrTaskNotFound
	}
	
	delete(l.tasks, title)
	return nil
}
```

Также вспомним,что мы хотим еще получить список всех невыполненных задач

list.go:
```go
func (l *List) ListNotCompletedTasks() map[string]Task {
	notCompletedTasks := make(map[string]Task, len(l.tasks))
	for k, v := range l.tasks {
		if !v.Completed {
			notCompletedTasks[k] = v
		}
	}
	return notCompletedTasks
}
```

Также создаем пустую мапу и копируем туда элементы,которые подходят по условию(тоесть,поля где Completed = false).

По сути,это все исправления,которые надо было сделать в первоначальном коде. Пока что этот ToDo List у нас на уровне репозитория. Репозиторий - сущность,ответственная за сохренение, поиск, изменение каких-то ресурсов.  В нашем случае ресурсом является задача и наш ToDo List как раз ответственен за то,чтобы получить эту задачу, отфильтровать задачу и тд. 

Пока что у нас уровень транспорта,сейчас объяснб подробнее. Вот у нас есть ToDo List,который может получать задачи, фильтровать их по параметру и тд, но чтобы он был досьупен по сети надо обернуть его в уровень транспорта(L4),чтобы мы могли посылать запросы и какие-то маниуляции над ToDo List'ом делать. По-хорошему надо еще уровни реализовать,но у нас тут REST API,а не архитектурные патерны. Повторюсь, у нас есть уровень репозитория(ToDo List) и надо обернуть его в уровень транспорта,чтобы была возможность посылать сетевые запросы. 

Далее создадим пакет http, и  первое,что мы там создадим-это файл  handlers.go. Там мы будем хранить хендлеры,которые будут обрабатывать запросы. И первым делом мы создаем структуру,где в качестве поля у нас будет указатель на ToDo List, чтобы над ним работать:
```go
gopackage http

import "rest/todo"

type HTTPHandlers struct {
	todoList *todo.List
}

func NewHTTPHandlers(todoList *todo.List) *HTTPHandlers {
	return &HTTPHandlers{
	todoList: todoList,
	}
}
```

Далее надо описывать каждый отдельный хендлер,каждый хендлер у нас будет методом HTTPHandlers{}:
```go
func NewHTTPHandlers(todoList *todo.List) *HTTPHandlers {
	return &HTTPHandlers{
		todoList: todoList,
	}
}

/*
pattern: /tasks
method: POST
info: JSON in HTTP Request body

succeed:
	- status code: 201 Created
	- responce body: JSON represent created task
failed:
	- status code: 400, 409, 500
	- responce body: JSON with error + time
*/
func (h *HTTPHandlers) HandleCreateNewTask(w http.ResponseWriter, r *http.Request) {
}

/*
pattern: /tasks/(title)
method: GET
info: pattern

succeed:
- status code: 200 OK
- response body: JSON represented found task
failed:
- status code: 400,404,500...
-response body: JSON with error + time
*/
func (h *HTTPHandlers) HandleGetTask(w http.ResponseWriter, r *http.Request) {
}

/*
pattern: /tasks
method: GET
info: ...

succeed:
	- status code: 200 OK
	- response body: JSON represented found tasks
failed:
	- status code: 400,500...
	-response body: JSON with error + time
*/
func (h *HTTPHandlers) HandleGetAllTasks(w http.ResponseWriter, r *http.Request) {
}

/*
pattern: /tasks?complited=false
method: GET
info: query params

succeed:
	- status code: 200 OK
	- response body: JSON represented found tasks
failed:
	- status code: 400,500...
	-response body: JSON with error + time
*/
func (h *HTTPHandlers) HandleGetAllUncomplitedTasks(w http.ResponseWriter, r *http.Request) {
}

/*
pattern: /tasks/(title)
method: PATCH
info: pattern + JSON in request body

succeed:
	- status code: 200 OK
	- response body: JSON represented found tasks
failed:
	- status code: 400,409,500...
	-response body: JSON with error + time
*/
func (h *HTTPHandlers) HandleCompleteTask(w http.ResponseWriter, r *http.Request) {

}

/*
pattern: /tasks/(title)
method: DELETE
info: pattern

succeed:
	- status code: 204 No Content
	- response body: ...

failed:
	- status code: 400,404,500...
	-response body: JSON with error + time
*/
func (h *HTTPHandlers) HandleDeleteTask(w http.ResponseWriter, r *http.Request) {
}
```

И того,опираясь на REST API, мы описали(пока в комментах),как хендлеры будут принимать данные,какой метод,статус-код использовать  в тех или иных ситуациях,и что да как возвращать.

Далее я создаю в дирректории http файл server.go, в нем будет описание самого сервера. Так как наш сервер зависит от хендлеров, то в качестве поля структуры HTTPServer мы указываем указатель на структуру HTTPServer{}. 

```go
type HTTPServer struct {
	httpHandlers *HTTPHandlers
}

func NewHTTPServer(httpHandlers *HTTPHandlers) *HTTPServer {
	return &HTTPServer{
		httpHandlers: httpHandlers,
	}
}
```

По классике пишем конструктор,помимо него еще и метод StartServer() для запуска сервера, но проблема в том,что у нас в хендлерах довольно сложные правила роутинга. Роутинг -  это про то,чтобы по входящим параметрам(патерн,метод,инфа) понять, какой хендлер нужно вызвать. Можно было бы по старинке это сделать через `http.HandleFunc()`, но этот метод не регулирует HTTP метод,нужный для того или иного хендлера, это первый момент. Второй момент заключается в том,что в каких-то патернах у нас зашита инфа,в каких-то нет. Третий момент -  у нас в каких-то запросах еще и query params ожидаются в патернах. Короче,если обрабатывать "по старинке" надо будет это все вручную обрабатывать через миллионы if-else, проблем в этом нет, но можно воспользоваться готовой библиотекой. 

Мы воспользуемся библиотекой `github.com/gorilla/mux`. Эта библиотека позволяет очень удробно организовывать роутер,для примера, согласование патерна и метода(прием в json будем организовывать сами) для хендлера HandleCreateNewTask() такое: `router.Path("/tasks").Methods("POST").HandlerFunc(s.httpHandlers.HandleCreateNewTask)`, тоесть мы указали,что хоти обрабатывать запрос на это хендлер,если у нас патер `/tasks` и метод POST.Ниже описание метода StartServer() (подробнее некоторые неописанные моменты описал в комментариях):
```go
func (s *HTTPServer) StartServer() error {
	router := mux.NewRouter() // рега роутера
	
	router.Path("/tasks").Methods("POST").HandlerFunc(s.httpHandlers.HandleCreateNewTask)
	router.Path("/tasks/{title}").Methods("GET").HandlerFunc(s.httpHandlers.HandleGetTask) // Так как в этом хендлере мы должны брать инфу их патерна,мы указываем аргумент,по какому параметру брать в {"инфа в патерне"}
	router.Path("/tasks").Methods("GET").HandlerFunc(s.httpHandlers.HandleGetAllTasks)
	router.Path("/tasks").Methods("GET").Queries("completed", "true").HandlerFunc(s.httpHandlers.HandleGetAllUncomplitedTasks) // так как этот хендлер работает с query params, мы вызываем Query() и там указываем первым значением ключ,вторым что ожидаем(если query params несколько,то указываем несколько пар ключ-значение)
	router.Path("/tasks/{title}").Methods("PATCH").HandlerFunc(s.httpHandlers.HandleCompleteTask)
	router.Path("/tasks/{title}").Methods("DELETE").HandlerFunc(s.httpHandlers.HandleDeleteTask)
	return http.ListenAndServe(":1488", router) // так как метод возвращает ошибку,то его и можно вернуть
}
```

Вот и описали сервер и роутер для него, осталось только описать хендлеры. 

Первый хендлер - HandleCreateTask(), так как он принимает в запросе JSON, нужна структура,в которой можно обработать тело JSON.Можно было бы взять структуру из ToDo,но проблема в том,что там много полей, а принять нам надо только заголовок и описание, поэтому как решение - DTO структура. DTO(data transfer object) -  структура, для передачи данных, тоесть структура,нужная просто для того,чтобы принять HTTP-запрос. 
```go
type TaskDTO struct {
	Title string
	Description string
}
```

Пока не описал работу HandleCreateTask(), хочу напомнить,что нам надо в ошибке передавать сообщение и врея совершения ошибки, для этого можно создать DTO структуру,но так как `http.Error()` принимает вторым аргументом строку,надо будет эту структуру преобразовть в строку, для этого пишем метод:
```go
type ErrorDTO struct {
	Message string
	Time time.Time
}

func (e ErrorDTO) ToString() string {
	b, err := json.MarshalIndent(e, "", " ") // превращает структуру в текст JSON формата
	if err != nil {
		panic(err)
	}
	return string(b)
}
```

Также,надо написать валидатор для TaskDTO{}(на пустые строки):
```go
func (t TaskDTO) ValidateForCreate() error {
	if t.Title == "" {
		return errors.New("title is empty")
	}
	
	if t.Description == "" {
		return errors.New("description is empty")
	}
	return nil
}
```

Вот как HandleCreateTask() выглядит в коде(в комментах опишу что да как):
```go
func (h *HTTPHandlers) HandleCreateNewTask(w http.ResponseWriter, r *http.Request) {
	var taskDTO TaskDTO // создаем экземпляр DTO структуры
	if err := json.NewDecoder(r.Body).Decode(taskDTO); err != nil { // запись параметров из тела запроса в taskDTO
		errDTO := ErrorDTO{
			Message: err.Error(),
			Time: time.Now(),
		}
		http.Error(w, errDTO.ToString(), http.StatusBadRequest)
		return // вот тут идет важный момент,мы создаем экземпляр структуры ErrorDTO, его полям присваиваем текст ошибки и время,когда выскочила ошибка, далее через http.Error(w, errDTO.ToString(), http.StatusBadRequest) передаем ошибку(второе поле это как раз структура переведенная в строку)
	}
	if err := taskDTO.ValidateForCreate(); err != nil {
		errDTO := ErrorDTO{
			Message: err.Error(),
			Time: time.Now(),
		}
		http.Error(w, errDTO.ToString(), http.StatusBadRequest)
		return // валидация на пустые значения 
	}
	
	todoTask := todo.NewTask(taskDTO.Title, taskDTO.Description) // создание таски и присваивание ей заголовка и описания из taskDTO
	if err := h.todoList.AddTask(todoTask); err != nil {
		errDTO := ErrorDTO{
			Message: err.Error(),
			Time: time.Now(),
		}
		if errors.Is(err, todo.ErrTaskAlreadyExists) {
			http.Error(w, errDTO.ToString(), http.StatusConflict)
			return
		} else {
			http.Error(w, errDTO.ToString(), http.StatusInternalServerError)
			return
		}
	} // тут мы добавляем таксу,НО так как у AddTask() возвращается error,надо понять,какой именно. Через errors.Is(err, todo.ErrTaskAlreadyExists) мы смотрим, принадлежит ли ошибка ErrTaskAlreadyExists(уже существует такая задача),если да,то возвращаем код 409, иначе возвращаем код 500 
	b, err := json.MarshalIndent(todoTask, "", " ")
	if err != nil {
		panic(err)
	}
	
	w.WriteHeader(http.StatusCreated)
	if _, err := w.Write(b); err != nil {
		fmt.Println("failed to write http response", err)
		return
	} // под конец мы переводит созданную таску в json формат, возвращаем код 201 и возвращаем его в теле ответа()
}
```

Далее описываем HandleGetTask(), там нам надо получить инфу из патерна, это можно сделать через библиотеку,которую мы импортировали для настройки роутера(`github.com/gorilla/mux`): `title := mux.Vars(r)["title"]`(тут раз мы получаем значение по ключу,то в теории можно было бы взять проверку на `ok`, но так как у нас в роутере настройка на то,что мы должны вызвать этот хендлер по title,то нет смысла проверять), описание хендлера ниже:
```go
func (h *HTTPHandlers) HandleGetTask(w http.ResponseWriter, r *http.Request) {
	title := mux.Vars(r)["title"] // тут мы делаем то,что я описал выше
	task, err := h.todoList.GetTask(title) // берем таску из туду листа и проверяем на ошибки
	if err != nil {
		errDTO := ErrorDTO{
			Message: err.Error(),
			Time: time.Now(),
		}
		if errors.Is(err, todo.ErrTaskNotFound) {
			http.Error(w, errDTO.ToString(), http.StatusNotFound)
			return
		} else {
			http.Error(w, errDTO.ToString(), http.StatusInternalServerError)
			return
		}
	}
	b, err := json.MarshalIndent(task, "", " ")
	if err != nil {
		panic(err)
	}
	w.WriteHeader(http.StatusOK)
	if _, err := w.Write(b); err != nil {
		fmt.Println("failed to write http response", err)
		return
	}
}
```

Далее надо описать хендлер для получения всех задач HandleGetAllTasks(). В этом хендлере не принимается инфа на входящий запрос, так что можно ее не принимать(да и не надо) и сразу брать все таски и возвращать их в теле ответа:
```go
	func (h *HTTPHandlers) HandleGetAllTasks(w http.ResponseWriter, r *http.Request) {
	tasks := h.todoList.ListTasks()
	b, err := json.MarshalIndent(tasks, "", " ")
	if err != nil {
	panic(err)
	}
	w.WriteHeader(http.StatusOK)
	if _, err := w.Write(b); err != nil {
	fmt.Println("failed to write http response", err)
	return
	}
}
```

Далее описываем хендлер для получения всех невыполненных задач HandleGetAllUncomplitedTasks(), мы уже в роутере описали то,что этот хендлер должен принимать query params с ключ=значение "complited" = "false", так что дополнительно это считывать не надо, код будет идентичен хендлеру HandleGetAllTasks():
```go
func (h *HTTPHandlers) HandleGetAllUncomplitedTasks(w http.ResponseWriter, r *http.Request) {
	uncomplitedTasks := h.todoList.ListUncompletedTasks()
	b, err := json.MarshalIndent(uncomplitedTasks, "", " ")
	if err != nil {
		panic(err)
	}
	w.WriteHeader(http.StatusOK)
	if _, err := w.Write(b); err != nil {
		fmt.Println("failed to write http response", err)
		return
	}
}
```

Дальше идет хендлер,который выполняет задачу, идентефикатор берем с патерна, а в самом хенндлере будет передано в запросе поле,которое надо поменять, значит,нужна еще одна DTO:
```go
type CompleteTaskDTO struct {
	Complete bool
}

func (h *HTTPHandlers) HandleCompleteTask(w http.ResponseWriter, r *http.Request) {
	var completeDTO CompleteTaskDTO 
	if err := json.NewDecoder(r.Body).Decode(&completeDTO); err != nil {
		errDTO := ErrorDTO{
			Message: err.Error(),
			Time: time.Now(),
		}
		http.Error(w, errDTO.ToString(), http.StatusBadRequest)
		return
	}
	
	title := mux.Vars(r)["title"] // берем заголовок из патерна
	
	if completeDTO.Complete { // в зависимости от принимаемого значения меняем в листе значение поля Completed в таске
		if err := h.todoList.CompleteTask(title); err != nil {
			errDTO := ErrorDTO{
				Message: err.Error(),
				Time: time.Now(),
			}
			if errors.Is(err, todo.ErrTaskNotFound) {
				http.Error(w, errDTO.ToString(), http.StatusNotFound)
				return
			} else {
				http.Error(w, errDTO.ToString(), http.StatusInternalServerError)
				return
			}
	} else {
		if err := h.todoList.UncompleteTask(title); err != nil {
		errDTO := ErrorDTO{
			Message: err.Error(),
			Time: time.Now(),
		}
		if errors.Is(err, todo.ErrTaskNotFound) {
			http.Error(w, errDTO.ToString(), http.StatusNotFound)
			return
		} else {
			http.Error(w, errDTO.ToString(), http.StatusInternalServerError)
		return
			}
			}
		}
	}
}
```

Далее описываем хендлер для удаления задач, тут единственное нам только код 204 должен вернуться,без тела запроса полноценного:
```go
func (h *HTTPHandlers) HandleDeleteTask(w http.ResponseWriter, r *http.Request) {
	title := mux.Vars(r)["title"]
	if err := h.todoList.DeleteTask(title); err != nil {
		errDTO := ErrorDTO{
			Message: err.Error(),
			Time: time.Now(),
		}
		if errors.Is(err, todo.ErrTaskNotFound) {
			http.Error(w, errDTO.ToString(), http.StatusNotFound)
			return
		} else {
			http.Error(w, errDTO.ToString(), http.StatusInternalServerError)
			return
		}
	}
	w.WriteHeader(http.StatusNoContent)
}
```

Вроде хендлеры мы описали,но тут нужно вспомнить про гонку данных... Так как мы работаем с todoList,он используется повсеместно,значит надо как-то избежать гонку данных. Чтобы в ручках не добавлять по куче мьютексов,можно на уровне ToDoList объявить свой мьютекс
```go
type List struct {
	tasks map[string]Task
	mtx sync.RWMutex
}
```
Почему RWMutex,а не обычный? Потому что у нас помимо нагрузки на запись есть еще нагрузка на чтение, так как у нас мало того кучу методов и ручек на чтение,так еще и юзер будет часто открывать список просматриваемых задач. Если посмотреть,то поле `tasks map[string]Task` у нас используется повсеместно, это разделяемый ресурс, который может привести к гонке данных. В теории,если рассмотреть метод AddTasks(), заизоилоровать это поле можно так:
```go
func (l *List) AddTask(task Task) error {
	l.mtx.RLock() // регулировка чтения
	if _, ok := l.tasks[task.Title]; ok {
		l.mtx.RUnlock() // тут нужна разблокировка,так как если ее не снимем,то горутина просто заблочитка и все упадет
		return ErrTaskAlreadyExists
	}
	l.mtx.RUnlock()
	
	l.mtx.Lock() // регулировка записи
	l.tasks[task.Title] = task
	l.mtx.Unlock()
	
	return nil

}
```

Но тут не все так просто, у нас может быть такой сценарий,когда 2 хендлера срабатывают одновременно,  и тут попдобная блокировка может просто не сработать. В данном случае,представим,что 2 горутины одновременно вызывают метод AddTask(), где у одного заголовок X, а у другого заголовок тоже X, обе горутины начинают проверять,есть ли задача с тем или иным заголовком, обе приходят к тому,что ее нет и оба перезаписывают одну и ту же мапу. Тут есть только 1 выход-на момент записи нужно гарантировать то,что состояние мапы на момент записи не изменилось, тоесть по сути, заблочить весь метож:
```go
func (l *List) AddTask(task Task) error {
	l.mtx.Lock()
	if _, ok := l.tasks[task.Title]; ok {
		l.mtx.Unlock()
		return ErrTaskAlreadyExists
	}
	l.tasks[task.Title] = task
	l.mtx.Unlock()
	return nil
}
```

Но так как отслеживать закрытие мбютекса зачастую бывает неудобно, можно воспользоваться стеком отложенных функций, и когда закончится подпрограмма можно оттуда взять функцию закрытия мьютекса
```go
/*
Пока только на примере AddTask(), так как я не вижу смысла сюда листить весь файл list.go, там где мы чисто читаем используем RW-операции, в противном случае берем обычную блокировку по мьютексу
*/
func (l *List) AddTask(task Task) error {
	l.mtx.Lock()
	defer l.mtx.Unlock() 
	if _, ok := l.tasks[task.Title]; ok {
		return ErrTaskAlreadyExists
	}
	l.tasks[task.Title] = task
	return nil
}
```

 Теперь у нас все готово, можно прописывать main.go:
 ```go
 package main

import (
	"fmt"
	"rest/http"
	"rest/todo"
)

func main() {
	todoList := todo.NewList()
	todoListHandlers := http.NewHTTPHandlers(todoList)
	todoHTTP := http.NewHTTPServer(todoListHandlers)
	if err := todoHTTP.StartServer(); err != nil {
		fmt.Println("Failed to start http server", err)
	} 
}
 ```

Тут нужно обратить внимание на оброботку ошибки,дело в том,что у нас в функции `http.ListenAndServe()` всегда возвращается non-nil error,тоесть,ошибка  будет при завершении работы и это будет `http.ErrServerClosed`

Чтобы это обработать,можно в ServerStart() сделать так:
```go
if err := http.ListenAndServe(":1488", router); err != nil {
	if errors.Is(err, http.ErrServerClosed) {
		return nil
	}
	return err
}
return nil
```

 Наше приложение написано,теперь надо его протестить(все тесты вставлять не буду,только особые кейсы)

Мы через метод POST создали таску по таким параметрам:
```json
{
    "title": "Домашка",
    "description": "курчас по мпс"
}
```

и нам в теле ответа вернулось такое 
```go
{
	"Title": "курчас по мпс",
	"Description": "Домашка",
	"Completed": false,
	"CreatedAt": "2026-09-14T12:16:59.047197417+03:00",
	"CompleteAt": null
}
```
Нас интересует только поле CompleteAt": null, точнее его значение null.
Что из себя представляет null? Он нам указывает на то,что значение для конкретного поля вообще не было указано. И не надо путать с nil, null это просто отсутствие осмысленного значение, а nil это пустой указатель

Когда тестировали хендлер,который отвечает за пометку задачи выполненной или невыполненной, выяснилось,что мы не прописали возвращение измененного элемента туду листа(нам возвращается 200,но в теле ответа ничего нет). И тут у нас проблема возникает,у нас хендлер на самом деле очень раздутый, и пока у нас что-то вернется из теле ответа, пройдет уйма времени и с этой таской может произойти что угодно. Во избежании этого можно чуток изменить методы структуры List{}, а именно прописать там возвращение задачи:
```go
func (l *List) CompleteTask(title string) (Task, error) {
	l.mtx.Lock()
	defer l.mtx.Unlock()
	task, ok := l.tasks[title]
	if !ok {
		return Task{}, ErrTaskNotFound
	}
	
	task.Complete()
	l.tasks[title] = task
	return l.tasks[title], nil
}

func (l *List) UncompleteTask(title string) (Task, error) {
	l.mtx.Lock()
	defer l.mtx.Unlock()
	task, ok := l.tasks[title]
	if !ok {
		return Task{}, ErrTaskNotFound
	}
	task.Uncomplete()
	l.tasks[title] = task
	return l.tasks[title], nil
}
```

Еще у нас проблема в том,что в хендлере,который меняет статус задачи, очень сильно дублируется код, это можго решить так:
```go
var (
	changedTask todo.Task
	err error
)


if completeDTO.Complete {
	changedTask, err = h.todoList.CompleteTask(title)
} else {
	changedTask, err = h.todoList.UncompleteTask(title)
}

if err != nil {
	errDTO := ErrorDTO{
	Message: err.Error(),
	Time: time.Now(),
	}
	if errors.Is(err, todo.ErrTaskNotFound) {
		http.Error(w, errDTO.ToString(), http.StatusNotFound)
	} else {
		http.Error(w, errDTO.ToString(), http.StatusInternalServerError)
	}
	return
}
```

Выше мы вынесли переменные таски и ошибки на уровень видимости всего хендлера,от чего мы избавтились от необходимости перепроверять код(теперь у нас везде только одна проверка ошибки), ну и под конец вернули в теле ответа измененную таску:
```go
b, err := json.MarshalIndent(changedTask, "", " ")
if err != nil {
	panic(err) // вообще,так делать не особо хорошо,так как структура может расшириться и могут быть проблемы, но пока,так какструктура маленькая,мы должны переводить все это без проблем
}
if _, err := w.Write(b); err != nil {
	fmt.Println("failed to write http response", err)
	return
}
```

Теперь у насдолжна возвращаться таска,где было изменено нужное нам поле.  