com.epic.taskmanager:
TaskmanagerApplication.java:
package com.epic.taskmanager;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@SpringBootApplication
public class TaskmanagerApplication {
	public static void main(String[] args) {
		SpringApplication.run(TaskmanagerApplication.class, args);
	}
}

com.epic.taskmanager.contoller:
TaskController.java:
package com.epic.taskmanager.contoller;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import com.epic.taskmanager.exceptions.TaskCollisionException;
import com.epic.taskmanager.model.Task;
import com.epic.taskmanager.service.TaskService;
@Controller
public class TaskController {	
	@Autowired
	private TaskServicetaskService;	
    @GetMapping("/")
	public String viewHomePage(Model model) {
		model.addAttribute("listTasks", taskService.getAllTasks());
		return "index";
	}	
    @GetMapping("/createNewTask")
    public String createNewTask(Model model) {
	Task task = new Task();
	model.addAttribute("task", task);
		return "new_task";  	
    }
    @PostMapping("/saveTask")
    public String saveTask(@ModelAttribute("task") Task task) throws TaskCollisionException {
	taskService.saveTask(task);
	return "redirect:/";
    }	
    @GetMapping("/updateTask/{id}")
    public String updateTask(@PathVariable(value = "id") int id, Model model){
	Task task = taskService.getTaskById(id);
	model.addAttribute("task", task);	
		return "update_task";	
    }
    @GetMapping("/deleteTask/{id}")
    public String deleteTask(@PathVariable (value = "id") int id) {
	this.taskService.deleteTaskById(id);
	return "redirect:/";
    }	
}

com.epic.taskmanager.exceptions:
ControllerExceptionHandler.java:
package com.epic.taskmanager.exceptions;
import java.sql.Date;
import java.util.ArrayList;
import java.util.List;
import javax.validation.ConstraintViolationException;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import org.springframework.web.context.request.WebRequest;
@RestControllerAdvice
public class ControllerExceptionHandler {
	@ExceptionHandler(MethodArgumentNotValidException.class)
	@ResponseStatus(value = HttpStatus.BAD_REQUEST)
	public ExceptionResponsehandleMethodArgumentNotValid(MethodArgumentNotValidExceptionex,WebRequest req) {
		List<String>inputErrors = new ArrayList<>();
		ex.getAllErrors().forEach(err -> {
			System.out.println(err.getDefaultMessage());
			inputErrors.add(err.getDefaultMessage());
		});
		ExceptionResponse error = new ExceptionResponse(new Date(0).toString(), HttpStatus.BAD_REQUEST.name(),inputErrors.toString(),req.getDescription(false));
	    return error;
	}	
	@ResponseStatus(value = HttpStatus.UNPROCESSABLE_ENTITY)
    @ExceptionHandler(ConstraintViolationException.class)
	public ExceptionResponsehandleConstraintViolationException(ConstraintViolationExceptionex,WebRequest req) {
		List<String>inputErrors = new ArrayList<>();
		ex.getConstraintViolations().forEach(err -> {
			System.out.println(err.getMessage());
			inputErrors.add(err.getMessage());
		});
		ExceptionResponse error = new ExceptionResponse(new Date(0).toString(), HttpStatus.BAD_REQUEST.name(), "Date should be future date only", req.getDescription(false));
	    return error;
    }
	@ExceptionHandler(TaskCollisionException.class)
	public ExceptionResponseTaskCollisionException(TaskCollisionExceptionex,WebRequest req) {
		ExceptionResponse error = new ExceptionResponse(new Date(0).toString(), HttpStatus.BAD_REQUEST.name(),"Task collision exception",req.getDescription(false));
	    return error;
	}
}

ExceptionResponse.java:
package com.epic.taskmanager.exceptions;
public class ExceptionResponse {
	String timestamp;
	String status;
	String error;
	String path;
	
	public ExceptionResponse() {}
	public ExceptionResponse(String timestamp, String status, String error, String path) {
		super();
		this.timestamp = timestamp;
		this.status = status;
		this.error = error;
		this.path = path;
	}
	public String getTimestamp() {
		return timestamp;
	}
	public void setTimestamp(String timestamp) {
		this.timestamp = timestamp;
	}
	public String getStatus() {
		return status;
	}
	public void setStatus(String status) {
		this.status = status;
	}
	public String getError() {
		return error;
	}
	public void setError(String error) {
		this.error = error;
	}
	public String getPath() {
		return path;
	}
	public void setPath(String path) {
		this.path = path;
	}
}

TaskCollisionException.java:
package com.epic.taskmanager.exceptions;
@SuppressWarnings("serial")
public class TaskCollisionException extends Exception{	
	@Override
	public String toString(){
	     return "Task Collision Exception";
	}
}

com.epic.taskmanager.model:
Task.java:
package com.epic.taskmanager.model;
import java.time.LocalDateTime;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.validation.constraints.Future;
import org.springframework.format.annotation.DateTimeFormat;
@Entity
public class Task {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private int taskId;
	private String taskDescription;
	@Future(message="Task Start time must be future only")
	@DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
	private LocalDateTimetaskStartTime;
	@Future(message="Task End time must be future only")
	@DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
	private LocalDateTimetaskEndTime;
	public int getTaskId() {
		return taskId;
	}
	public void setTaskId(int taskId) {
		this.taskId = taskId;
	}
	public String getTaskDescription() {
		return taskDescription;
	}
	public void setTaskDescription(String taskDescription) {
		this.taskDescription = taskDescription;
	}
	public LocalDateTimegetTaskStartTime() {
		return taskStartTime;
	}
	public void setTaskStartTime(LocalDateTimetaskStartTime) {
		this.taskStartTime = taskStartTime;
	}
	public LocalDateTimegetTaskEndTime() {
		return taskEndTime;
	}
	public void setTaskEndTime(LocalDateTimetaskEndTime) {
		this.taskEndTime = taskEndTime;
	}}

com.epic.taskmanager.repository:
TaskRepository.java:
package com.epic.taskmanager.repository;
import java.time.LocalDateTime;
import java.util.List;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;
import com.epic.taskmanager.model.Task;
@Repository
public interface TaskRepository extends JpaRepository<Task, Integer> {
	List<Task>findByTaskStartTimeBetweenOrTaskEndTimeBetween(LocalDateTimetaskStartTime,LocalDateTimetaskEndTime,
			LocalDateTimetaskStartingTime,LocalDateTimetaskEndingTime);
}

com.epic.taskmanager.service:
TaskService.java:
package com.epic.taskmanager.service;
import java.util.List;
import com.epic.taskmanager.exceptions.TaskCollisionException;
import com.epic.taskmanager.model.Task;
public interface TaskService {
  List<Task>getAllTasks();
  void saveTask(Task task) throws TaskCollisionException;
  public Task getTaskById(int id);
  public void deleteTaskById(int id);  
}



TaskServiceImpl.java:
package com.epic.taskmanager.service;
import java.util.List;
import java.util.Optional;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import com.epic.taskmanager.exceptions.TaskCollisionException;
import com.epic.taskmanager.model.Task;
import com.epic.taskmanager.repository.TaskRepository;
import com.epic.taskmanager.validator.Validations;
@Service
public class TaskServiceImpl implements TaskService{
	@Autowired
	private Validations validations;
	@Autowired
	private TaskRepositorytaskRepository;
	@Override
	public List<Task>getAllTasks() { 	
		return taskRepository.findAll();
	}
	@Override
	public void saveTask(Task task) throws TaskCollisionException {
	if(!validations.checkCollision(task.getTaskStartTime(),task.getTaskEndTime())) {
	this.taskRepository.save(task);	
		}
	}
	@Override
	public Task getTaskById(int id){
        Optional<Task> optional = taskRepository.findById(id);
        Task task = null;
        if(optional.isPresent()) {
	task = optional.get();
        }
		return task;
	}
	@Override
	public void deleteTaskById(int id) {
	this.taskRepository.deleteById(id);		
	}
}

com.epic.taskmanager.validator:
Validations.java:
package com.epic.taskmanager.validator;
import java.time.LocalDateTime;
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import com.epic.taskmanager.exceptions.TaskCollisionException;
import com.epic.taskmanager.model.Task;
import com.epic.taskmanager.repository.TaskRepository;
@Component
public class Validations {
	@Autowired
	TaskRepositorytaskRepository;
	boolean value;
	public booleancheckCollision(LocalDateTimetaskStartTime, LocalDateTimetaskEndTime) throws TaskCollisionException
	{
		value = false;
		List<Task>tasksList = taskRepository.findByTaskStartTimeBetweenOrTaskEndTimeBetween(taskStartTime, taskEndTime,
				taskStartTime, taskEndTime);
		if (!tasksList.isEmpty()) {
			value = true;
			throw new TaskCollisionException();
		}
		return value;
	}
}

index.html:
<!DOCTYPE html>
<html lang = "en" xmlns:th = "http://www.thymeleaf.org">
<head>
<meta charset="ISO-8859-1">
<title>Task Manager</title>
<link rel = "stylesheet"
href = "https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css"
  integrity = "sha384-MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO"
crossorigin = "anonymus">
</head>
<body>
<div class = "container my-2">
<h1>Task List</h1>
<a th:href = "@{/createNewTask}" class = "btnbtn-primary btn-sm mb-3"> Add Task</a>
<table border = "1" class = "table table-striped table-responsive-md">
<thead>
<tr>
<th> Task ID </th>
<th> Task name </th>
<th> Task Start Time </th>
<th> Task End Time </th>
<th> Actions </th>
</tr>
</thead>
<tbody>
<tr th:each="task : ${listTasks}">
<td th:text="${task.taskId}" >
<td th:text="${task.taskDescription}" >
<td th:text="${task.taskStartTime}" >
<td th:text="${task.taskEndTime}" >
<td><a th:href = "@{/updateTask/{id}(id=${task.taskId})}" class = "btnbtn-primary">Update</a>
<a th:href = "@{/deleteTask/{id}(id=${task.taskId})}" class = "btnbtn-danger">Delete</a>
</td>
</tr>
</tbody>
</table>
</div>
</body>
</html>

new_task.html:
<!DOCTYPE html>
<html lang = "en" xmlns:th = "http://www.thymeleaf.org">
<head>
<meta charset="ISO-8859-1">
<title>Task Manager</title>
<link rel = "stylesheet"
href = "https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css"
  integrity = "sha384-MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO"
crossorigin = "anonymus">
</head>
<body>
<div class = "container">
<h1> Task Manager</h1>
<hr>
<h2> Save Task</h2>
<form action = "#" th:action = "@{/saveTask}" th:object = "${task}" method = "POST">
<input type = "text" th:field = "*{taskDescription}" placeholder = "Task Desrciption" class = "form-control mb-4 col-4">
<input type = "text" th:field = "*{taskStartTime}" placeholder = "Task Start Time" class = "form-control mb-4 col-4">
<input type = "text" th:field = "*{taskEndTime}" placeholder = "Task End Time" class = "form-control mb-4 col-4">
<button type = "submit" class = "btnbtn-info col-2"> Save Task</button>
</form>
</div>
</body>
</html>

update_task.html:
<!DOCTYPE html>
<html lang = "en" xmlns:th = "http://www.thymeleaf.org">
<head>
<meta charset="ISO-8859-1">
<title>Task Manager</title>
<link rel = "stylesheet"
href = "https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css"
  integrity = "sha384-MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO"
crossorigin = "anonymus">
</head>
<body>
<div class = "container">
<h1> Task Manager</h1>
<hr>
<h2> Update Task</h2>
<form action = "#" th:action = "@{/saveTask}" th:object = "${task}" method = "POST">
<input type = "text" th:field = "*{taskId}" placeholder = "Task Id" readonly class = "form-control mb-4 col-4">
<input type = "text" th:field = "*{taskDescription}" class = "form-control mb-4 col-4">
<input type = "text" th:field = "*{taskStartTime}" class = "form-control mb-4 col-4">
<input type = "text" th:field = "*{taskEndTime}" class = "form-control mb-4 col-4">
<button type = "submit" class = "btnbtn-info col-2"> Update Task</button>
</form>
</div>
</body>
</html>
	
