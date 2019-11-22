<template>
    <div style="margin:10px">
        <h2>Student Performance</h2>
        <div class="form-inline" role="form" style="flex-flow:row nowrap">
            <div class="form-group mb1">
                <label v-bind:for="'name'" class="sr-only">Name</label>
                <input v-bind:type="'text'" maxlength="20" v-bind:id="'name'" v-bind:name="'name'" class="form-control" v-bind:class="getValidationClass('name')" v-bind:placeholder="'Enter Name'" v-model="formData['name']"/>
            </div>
            <div class="form-group mb2" v-for="(subject,index) in subjects" v-bind:key="index">
                <label v-bind:for="subject" class="sr-only">{{subject}}</label>
                <input v-bind:type="'text'" maxlength="1" v-bind:id="subject" v-bind:name="subject" class="form-control" v-bind:class="getValidationClass(subject)" v-bind:placeholder="'Enter '+subject" v-model="formData[subject]"/>
            </div>
            <div class="form-group mb1">
                <button type="button" :disabled='!isFormValid()' class="btn btn-primary" @click="addStudent">Add Student</button>
            </div>        
        </div>
        <table class="table table-striped table-dark">
        <thead>
            <tr>
            <th scope="col">#</th>
            <th scope="col">Name</th>
            <th scope="col" v-for="(subject,index) in subjects" v-bind:key="index">{{subject}}</th>
            <th scope="col">GPA</th>
            <th></th>
            </tr>
        </thead>
        <tbody>
            <tr v-for="(student,index) in students" v-bind:key="student._id" v-bind:class="getBestWorstClasses(student)">
            <th scope="row">{{index+1}}</th>
            <td>{{student.name}}</td>
            <td scope="row" v-for="(subject,index) in subjects" v-bind:key="index">{{student.gradeMap[subject]}}</td>
            <td>{{student.gpa}}</td>
            <td><button @click="removeStudent(student._id)">Remove</button></td>
            </tr>
        </tbody>
        </table>
    </div>
</template>

<style>
    tr.best-gpa > td, tr.best-gpa > th {
        color: black;
    }

    tr.worst-gpa > td, tr.worst-gpa > th {
        color:black;
    }

    .form-inline .form-group {
        flex: 1 1 auto;
        margin:10px;
    }

    .form-inline .form-control {
        width: 100%;
    }

    .form-group button {
        white-space: nowrap;
    }

    @media (min-width: 576px) {
         .form-inline .form-control {
            width: 100%;
        }   
    }
</style>

<script>

let extractGrade = (gradeDescription) => {
          let gradeMap = {
              A: 4,
              B: 3,
              C: 2,
              D: 1,
              F: 0 
          }
          let index = gradeDescription.length - 1
          return gradeMap[gradeDescription.charAt(index)]
      }

let calculateGPA = (grades) => {
    let total = 0;
    grades.forEach( grade => {
        total += extractGrade(grade);
    })
    return total / grades.length
}

let getGradeMap = (grades) => {
          let map = {}
          grades.forEach( gradeDesc => {
            const end = gradeDesc.indexOf(" ")
            const subject = gradeDesc.substring(0,end)
            const grade = gradeDesc.charAt(gradeDesc.length - 1)
            map[subject] = grade
          })

          return map
        //   return {
        //       Math: 'A',
        //       History: 'B',
        //       Science: 'A',
        //       English: 'B'
        //   }
}

let toSimplifiedStudent = (student) => {
    return {
        _id: student._id,
        name: student.name,
        gpa: calculateGPA(student.grades),
        gradeMap: getGradeMap(student.grades)
    }
}

export default {
  name: "StudentInfo",
  methods: {
      getBestWorstClasses: function(student) {
          if(this.isBestGPA(student.gpa) && this.isWorstGPA(student.gpa)) {
              return ''
          } else if(this.isBestGPA(student.gpa)) {
             return 'best-gpa bg-success'
          } else if (this.isWorstGPA(student.gpa)){
             return 'worst-gpa bg-danger'
          }
        
      },
      getValidationClass: function(column) {
        return !this.isPristine(this.formData[column]) ? (this.validateInputType(column, this.formData[column]) ? 'is-valid' : 'is-invalid') : ''
      },
      isPristine: function(value) {
          return value == null
      },
      isValidGrade: function(value) {
          return ['A','B','C','D','F'].indexOf(value.toUpperCase()) > -1
      },
      isValidName: function(value) {
          return value !== ''
      },
      validateInputType : function(key, value) {
        return key === "name" ? this.isValidName(value) : this.isValidGrade(value)
      },
      isFormValid: function() {
          let isValid = true;
          
          ["name"].concat(this.subjects).forEach( key => {
                let value = this.formData[key]
                isValid = isValid && !this.isPristine(value) && this.validateInputType(key, value)
          })
          return isValid
      },
      isBestGPA: function(gpa) {
          return gpa === this.bestGPA
      },
      isWorstGPA: function(gpa) {
          return gpa === this.worstGPA
      },
      resetBestAndWorst: function(){
          this.bestGPA = 0
          this.worstGPA = 4
      },
      persistBestAndWorst: function(gpa) {
        if(gpa > this.bestGPA ) {
            this.bestGPA = gpa
        }
        if(gpa < this.worstGPA ) {
            this.worstGPA = gpa
        }
      },
      makeStudent: function(formData) {
          const formattedGrades = this.subjects.map( subject => `${subject} - ${formData[subject].toUpperCase()}`)
          return {
              _id : this.students.length + 1,
              name: formData.name,
              gpa: calculateGPA(formattedGrades),
              gradeMap :getGradeMap(formattedGrades)
          }
      },
      addStudent: function() {
          const student = this.makeStudent(this.formData)
          this.students.push(student)
          this.persistBestAndWorst(student.gpa)
      },
      removeStudent: function(id) {
          this.students = this.students.filter(student => student._id !== id)
          this.resetBestAndWorst()
          this.students.map( student => this.persistBestAndWorst(student.gpa) )
      }
  },
  data: () => ({
    formData: {name:null, Math: null, History: null, Science: null, English: null},
    subjects: ['Math', 'History', 'Science', 'English'],
    bestGPA: 0,
    worstGPA: 4,
    students: [
      {
        _id: 1,
        name: "Adam",
        grades: ["Math - A", "History - B", "Science - A", "English - B"],
        img: "https://i.pravatar.cc/300",
        gender: "M",
        birthday: "July 23, 2003",
        athlete: true,
        grade: 10
      },
      {
        _id: 2,
        name: "John",
        grades: ["Math - B", "History - B", "Science - B", "English - B"],
        img: "https://i.pravatar.cc/300",
        gender: "M",
        birthday: "September 8, 2005",
        athlete: false,
        grade: 8
      },
      {
        _id: 3,
        name: "Sarah",
        grades: ["Math - C", "History - B", "Science - B", "English - C"],
        img: "https://i.pravatar.cc/300",
        gender: "F",
        birthday: "January 17, 2004",
        athlete: false,
        grade: 9
      },
      {
        _id: 4,
        name: "Katie",
        grades: ["Math - D", "History - B", "Science - C", "English - B"],
        img: "https://i.pravatar.cc/300",
        gender: "F",
        birthday: "October 6, 2005",
        athlete: true,
        grade: 8
      },
      {
        _id: 5,
        name: "Phil",
        grades: ["Math - F", "History - B", "Science - C", "English - D"],
        img: "https://i.pravatar.cc/300",
        gender: "M",
        birthday: "August 13, 2001",
        athlete: false,
        grade: 12
      }
    ].map( student => toSimplifiedStudent(student))
  }),
  created: function () {
    this.students.forEach( student => this.persistBestAndWorst(student.gpa))
  },
  props: {}
};
</script>

<style scoped>
</style>