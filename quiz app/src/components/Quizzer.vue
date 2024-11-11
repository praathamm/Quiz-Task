<template>
   <div id="main-container">
     <div v-if="!quizStarted" id="difficulty-section">
       <h1>Welcome to the Quiz!</h1>
       <input 
         type="text" 
         v-model="user" 
         placeholder="Enter your name" 
         required 
         id="user-input" 
       />
       <div id="difficulty-buttons">
         <button @click="startQuiz('easy')" class="difficulty-btn">Easy</button>
         <button @click="startQuiz('medium')" class="difficulty-btn">Medium</button>
         <button @click="startQuiz('hard')" class="difficulty-btn">Hard</button>
       </div>
     </div>
 
     <div v-if="quizStarted">
       <div class="timer" v-if="quizStarted">
         Time: {{ timer }}
       </div>
       <div id="score-container" v-if="visibility2">
         Your Current Score: {{ score }}
       </div>
       <h2 id="question-container" v-if="visibility2">
         {{ question.q }}
       </h2>
       <div id="options-container" v-if="visibility2">
         <button @click="checkAnswer(question.o1)" class="answer-btn" id="option1">
           {{ question.o1 }}
         </button>
         <button @click="checkAnswer(question.o2)" class="answer-btn" id="option2">
           {{ question.o2 }}
         </button>
         <button @click="checkAnswer(question.o3)" class="answer-btn" id="option3">
           {{ question.o3 }}
         </button>
         <button @click="checkAnswer(question.o4)" class="answer-btn" id="option4">
           {{ question.o4 }}
         </button>
       </div>
       <button v-if="visibility" @click="nextQuestion" class="next-btn" id="next-button">
         Next Question
       </button>
       <button v-if="visibility2" @click="skipQuestion" class="next-btn" id="skip-button">
         Skip Question
       </button>
       <div id="end-screen" v-if="!visibility2" class="next-btn">
         <div class="final-score-container">
           <h1 class="final-score-heading">Thank you for playing!</h1>
           <p>Your final score was <span class="score-value">{{ score }}</span></p>
           <p>Questions Attempted: <span class="score-value">{{ attempted }}</span></p>
           <p>Questions Skipped: <span class="score-value">{{ skipped }}</span></p>
           <p>Time Taken: <span class="score-value">{{ timeTaken }} seconds</span></p>
           <input
             type="text"
             v-model="user"
             @change="namesaver"
             placeholder="Enter your name"
             required
             id="user-input"
           />
           <br />
           <button @click="tableshower" id="show-scores-btn">Show previous scores</button>
         </div>
       </div>
       <div id="score-table-container" v-if="showTable">
         <table border="1" id="score-table">
           <thead>
             <tr>
               <th>Name</th>
               <th>Score</th>
               <th>Attempted</th>
               <th>Skipped</th>
               <th>Time</th>
             </tr>
           </thead>
           <tbody>
             <tr v-for="(userData, index) in users" :key="index">
               <td>{{ userData.name }}</td>
               <td>{{ userData.score }}</td>
               <td>{{ userData.attempted }}</td>
               <td>{{ userData.skipped }}</td>
               <td>{{ userData.time }} seconds</td>
             </tr>
           </tbody>
         </table>
       </div>
     </div>
   </div>
 </template>
 
 <script>
 import axios from "axios";
 
 export default {
   data() {
     return {
       question: {},
       selectedAnswer: "",
       visibility: false,
       visibility2: true,
       score: 0,
       attempted: 0,
       skipped: 0,
       norepeat: [],
       showTable: false,
       user: "",
       users: [],
       quizStarted: false,
       currentDifficulty: "easy", 
       questionBaseUrl: "http://localhost:3000/question", 
       timer: 0,
       timeTaken: 0,
       timerInterval: null,
     };
   },
   methods: {
     async fetchQuestion() {
       const response = await axios.get(this.questionBaseUrl);
       let i = Math.floor(Math.random() * 10);
       while (this.norepeat.includes(i)) {
         i = Math.floor(Math.random() * 10);
       }
       this.norepeat.push(i);
       this.question = response.data[i];
       this.selectedAnswer = "";
     },
     checkAnswer(selectedOption) {
       this.selectedAnswer = selectedOption;
       if (selectedOption === this.question.ans) {
         this.score++;
       }
       this.visibility = true;
     },
     namesaver() {
       const userData = {
         name: this.user,
         score: this.score,
         attempted: this.attempted,
         skipped: this.skipped,
         time: this.timeTaken,
       };
       axios
         .post("http://localhost:3000/user", userData)
         .then((response) => {
           console.log("User data saved successfully:", response.data);
         })
         .catch((error) => {
           console.error("Error saving user data:", error);
         });
     },
     tableshower() {
       this.showTable = !this.showTable;
       if (this.showTable) {
         this.fetchUsers();
       }
     },
     async fetchUsers() {
       try {
         const response = await axios.get("http://localhost:3000/user");
         this.users = response.data;
       } catch (error) {
         console.error("Error fetching users' data:", error);
       }
     },
     nextQuestion() {
       if (this.norepeat.length > 5) {
         this.visibility2 = false;
         this.visibility = false;
       }
       this.attempted++;
       this.fetchQuestion();
     },
     skipQuestion() {
       if (this.norepeat.length > 5) {
         this.visibility2 = false;
         this.visibility = false;
       }
       this.skipped++;
       this.fetchQuestion();
     },
     startQuiz(difficulty) {
       this.currentDifficulty = difficulty;
       this.quizStarted = true;
       this.timeTaken = 0;
       this.timer = 0;
 
       if (difficulty === "easy") {
         this.questionBaseUrl = "http://localhost:3000/question"; 
       } else if (difficulty === "medium") {
         this.questionBaseUrl = "http://localhost:3000/question2"; 
       } else if (difficulty === "hard") {
         this.questionBaseUrl = "http://localhost:3000/question3"; 
       }
 
       this.fetchQuestion();
       
       this.startTimer();
     },
     startTimer() {
    if (this.timerInterval) return; 
    this.timerInterval = setInterval(() => {
      this.timer++;
      this.timeTaken = this.timer;
    }, 1000);
  },
  stopTimer() {
    clearInterval(this.timerInterval);
    this.timerInterval = null; 
  },
     endQuiz() {
       this.stopTimer();
       this.namesaver();
     },
   },
   mounted() {
     this.fetchQuestion();
   },
   beforeDestroy() {
     if (this.timerInterval) {
       clearInterval(this.timerInterval);
     }
   },
 };
 </script>
 
 <style scoped>
 * {
   font-family: 'Georgia', serif;
 }
 
 #main-container {
   background: linear-gradient(135deg, #2a74d9, #1565c0);
   color: white;
   padding: 50px;
   text-align: center;
   margin: 0;
   font-size: 2em;
 }
 
 #difficulty-section {
   text-align: center;
   margin-top: 50px;
 }
 
 #difficulty-section h1 {
   font-size: 2.5em;
   margin-bottom: 20px;
 }
 
 .difficulty-btn {
   background-color: #2a74d9;
   color: white;
   font-size: 1.8em;
   padding: 20px;
   margin: 10px;
   border-radius: 15px;
   cursor: pointer;
   width: 200px;
   transition: background-color 0.3s;
 }
 
 .difficulty-btn:hover {
   background-color: #1565c0;
 }
 
 #user-input {
   font-size: 1.5em;
   padding: 10px;
   margin-top: 20px;
   width: 60%;
   border-radius: 5px;
   border: none;
 }
 
 .timer {
   position: absolute;
   top: 20px;
   right: 50px;
   font-size: 2em;
   color: white;
 }
 
 #score-container {
   margin-bottom: 30px;
 }
 
 #question-container {
   font-size: 2.5em;
   margin-bottom: 30px;
 }
 
 #options-container {
   margin-bottom: 50px;
 }
 
 .answer-btn {
   background-color: #2a74d9;
   color: white;
   font-size: 1.8em;
   padding: 20px;
   border-radius: 15px;
   border: none;
   cursor: pointer;
   transition: transform 0.3s ease;
   margin: 10px;
   width: 80%;
 }
 
 .answer-btn:hover {
   transform: scale(1.1);
   background-color: #1565c0;
 }
 
 #next-button, #skip-button {
   background-color: #1565c0;
   color: white;
   padding: 20px;
   font-size: 1.8em;
   border: none;
   margin-top: 30px;
   border-radius: 15px;
   cursor: pointer;
   width: 80%;
 }
 
 #end-screen {
   margin-top: 40px;
 }
 
 .final-score-container {
   background: rgba(0, 0, 0, 0.6);
   padding: 30px;
   border-radius: 15px;
   box-shadow: 0 0 15px rgba(0, 0, 0, 0.3);
   font-size: 1.8em;
 }
 
 .final-score-heading {
   font-size: 2.5em;
   color: #FFDD57;
 }
 
 .score-value {
   font-weight: bold;
   color: #FFDD57;
 }
 
 #user-input {
   font-size: 1.5em;
   padding: 10px;
   width: 60%;
   margin-top: 20px;
   border-radius: 5px;
   border: none;
 }
 
 #show-scores-btn {
   background-color: #1565c0;
   color: white;
   padding: 20px;
   font-size: 1.5em;
   border: none;
   margin-top: 20px;
   border-radius: 15px;
   cursor: pointer;
 }
 
 #score-table-container {
   margin-top: 50px;
 }
 
 #score-table {
   width: 100%;
   border-collapse: collapse;
   margin-top: 50px;
 }
 
 #score-table th, #score-table td {
   padding: 15px;
   text-align: center;
   border: 2px solid white;
   font-size: 1.5em;
 }
 
 #score-table th {
   background-color: #2a74d9;
 }
 
 #score-table td {
   background-color: #1565c0;
 }
 </style>
 