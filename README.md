# StudentDetails
package net.guides.springboot.springbootcrudrestapivalidation.controller;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

import javax.validation.Valid;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Sort;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import net.guides.springboot.springbootcrudrestapivalidation.exception.ResourceNotFoundException;
import net.guides.springboot.springbootcrudrestapivalidation.model.AgeResponse;
import net.guides.springboot.springbootcrudrestapivalidation.model.GradeResponse;
import net.guides.springboot.springbootcrudrestapivalidation.model.Student;
import net.guides.springboot.springbootcrudrestapivalidation.repository.StudentRepository;
import net.guides.springboot.springbootcrudrestapivalidation.response.ResponseHandler;

@RestController
@RequestMapping("/api/v1")
public class StudentController {
	@Autowired
	private StudentRepository studentRepository;


	//for getting Details search by name
	// Burl="http://localhost:8089/api/v1/getDetailsByName?Name=";
	@GetMapping(value="/getDetailsByName")
	public ResponseEntity<Student> getStudentByName(@RequestParam("Name") String studentName)
			throws ResourceNotFoundException {
		Student student = studentRepository.findByName(studentName)
				.orElseThrow(() -> new ResourceNotFoundException("student not found for this name :: " + studentName));
		return ResponseEntity.ok().body(student);
	}


	//for making a post request
	//url="http://localhost:8089/api/v1/AddUser";
	@PostMapping( "/AddUser")
	public ResponseEntity<Object> Post( @RequestBody Student student) {
		studentRepository.save(student);
		try {
			Student std = StudentRepository.Post(student);
			return ResponseHandler.generateResponse("Successfully added data!", HttpStatus.OK, std);
		} catch (Exception e) {

			return ResponseHandler.generateResponse(e.getMessage(), HttpStatus.MULTI_STATUS,null);
		}

	}



	//for updating the record for the selected id
	//url="http://localhost:8089/api/v1/UpdateUser?id="
	@PutMapping("/UpdateUser")
	public ResponseEntity<Student> updateStudent(@RequestParam("id") Long studentId,
			@Valid @RequestBody Student studentDetails) throws ResourceNotFoundException {
		System.out.println("");
		Student student = studentRepository.findById(studentId)
				.orElseThrow(() -> new ResourceNotFoundException("Student not found for this id :: " + studentId));
		student.setName(studentDetails.getName());
		student.setGrade(studentDetails.getGrade());
		student.setAge(studentDetails.getAge());

		final Student updatedStudent = studentRepository.save(student);
		return ResponseEntity.ok(updatedStudent);
	}



	//fo deleting the record for that particular id only
	//	url="http://localhost:8089/api/v1/DeleteUser?name="
	@DeleteMapping("/DeleteUser")
	public Map<String, Boolean> deleteStudent(@RequestParam("name") String studentName)
			throws ResourceNotFoundException {
		Student student = studentRepository.findByName(studentName)
				.orElseThrow(() -> new ResourceNotFoundException("Student not found for this id :: " + studentName));
		studentRepository.delete(student);
		Map<String, Boolean> response = new HashMap<>();
		response.put("deleted", Boolean.TRUE);
		return response;
	}


	//for getting the grade of students in order by 
	//url=http://localhost:8089/api/v1/GetUserOrderByGrade?grade=b
	@GetMapping("/GetUserOrderByGrade")
	public GradeResponse getGrade(@RequestParam("grade") String grade){
		System.out.println(grade);
		List<Student> studentList= studentRepository.tempQuery(grade);
		GradeResponse graderesponse=new GradeResponse();
		graderesponse.setGrade(grade);
		graderesponse.setListOfStudent(studentList);
		return graderesponse;

	}


	//for getting request order by name
	//url=http://localhost:8089/api/v1/GetUserOrderByName
	@GetMapping("/GetUserOrderByName")
	public List<Student> getAllStudent() {
		return studentRepository.findAll(Sort.by(Sort.Direction.ASC, "name"));
	}


	//for getting request order by age
	//url=http://localhost:8089/api/v1/GetUserOrderByAge?age=
	@GetMapping("/GetUserOrderByAge")
	public AgeResponse getAge(@RequestParam("age") String age){
		System.out.println(age);
		List<Student> studentList= studentRepository.ageQuery(age);
		AgeResponse ageresponse=new AgeResponse();
		ageresponse.setAge(age);
		ageresponse.setListOfStudent(studentList);
		return ageresponse;

	}
}
