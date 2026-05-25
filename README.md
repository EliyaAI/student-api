# student-api
#A simple FastAPI project for managing students with CRUD operations.

from fastapi import FastAPI, HTTPException, Query

from typing import Optional, List

from pydantic import BaseModel, Field

from fastapi.responses import HTMLResponse


app = FastAPI()



class Student(BaseModel):

    name: str = Field(min_length=3, max_length=50)

    age: int = Field(gt=15, lt=100)

    course: str

    score: float = Field(ge=0, le=100)

    passed: bool = False


# create student
next_id = 1
students = []

@app.post("/students")
def create_student(student: Student):
    global next_id

    student_data = student.dict()
    student_data["id"] = next_id
    next_id += 1

    students.append(student_data)

    return {
        "message": "Student created successfully",
        "student": student_data
    }


# get students and filter
@app.get("/students")
def get_students(

    passed: Optional[bool] = None,
    course: Optional[str] = None

):

    result = students

    if passed is not None:
        result = [s for s in result if s["passed"] == passed]

    if course:
        result = [s for s in result if s["course"].lower() == course.lower()]

    return result


# get student with id
@app.get("/students/{student_id}")
def get_student(student_id: int):

    for student in students:

        if student["id"] == student_id:
            return student

    raise HTTPException(
        status_code=404,
        detail="Student not found"
    )


# update
@app.put("/students/{student_id}")
def update_student(student_id: int, updated_student: Student):

    for student in students:

        if student["id"] == student_id:

            student["name"] = updated_student.name
            student["age"] = updated_student.age
            student["course"] = updated_student.course
            student["score"] = updated_student.score
            student["passed"] = updated_student.passed

            return {
                "message": "Student updated",
                "student": student
            }

    raise HTTPException(
        status_code=404,
        detail="Student not found"
    )

