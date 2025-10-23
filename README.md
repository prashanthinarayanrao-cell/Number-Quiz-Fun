# Number-Quiz-Fun
Number Quiz Fun is an engaging web-based learning game for Grade 1 students that teaches natural numbers through colorful visuals, fun questions, and voice narration. It helps children easily understand what comes before, after, and between numbers while enjoying playful learning.
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Number Quiz Fun!</title>
  <style>
    body { font-family: 'Comic Sans MS', cursive, sans-serif; text-align: center; margin: 0; background: linear-gradient(135deg,#fce4ec,#e3f2fd); min-height:100vh; display:flex; align-items:center; justify-content:center; }
    .quiz-container { background:#ffffffcc; padding:40px 35px; border-radius:25px; box-shadow:0 10px 25px rgba(0,0,0,0.15); width:460px; max-width:94vw; transition: transform 0.4s ease; }
    .quiz-container:hover { transform: scale(1.03); }
    h1 {
      margin:0 0 16px 0;
      font-size:3.5rem;
      font-weight:bold;
      color:#ff4081;
      text-shadow: 3px 3px 6px rgba(0,0,0,0.2);
      background: none;
    }
    h2{margin:8px 0 18px 0;color:#1976d2;font-size:1.6rem;}
    .options{display:flex;flex-direction:column;gap:12px;margin-top:6px}
    .option-btn{padding:14px 16px;font-size:22px;border-radius:14px;border:none;color:#fff;background:linear-gradient(135deg,#ff8a65,#f06292);cursor:pointer;transition:transform .12s ease,box-shadow .12s ease;width:100%;box-shadow:0 6px 12px rgba(0,0,0,0.08)}
    .option-btn.selected{background:linear-gradient(135deg,#81c784,#4caf50);transform:scale(1.03);box-shadow:0 10px 18px rgba(0,0,0,0.16)}
    .option-btn.correct{background:linear-gradient(135deg,#66bb6a,#43a047);}
    .option-btn.hint{box-shadow:0 8px 18px rgba(102,187,106,0.45)}
    #submitButton,#nextButton{margin-top:24px;padding:14px 26px;font-size:20px;border-radius:999px;border:none;color:#fff;cursor:pointer;transition:transform 0.3s ease;}
    #submitButton:hover,#nextButton:hover{transform:scale(1.05)}
    #submitButton{background:linear-gradient(45deg,#ff4081,#f48fb1)}
    #nextButton{background:linear-gradient(45deg,#2196f3,#64b5f6);display:none}
    .result{margin-top:16px;font-size:1.2rem;min-height:48px}
    #score{margin-top:12px;font-size:1rem;color:#1976d2;}
  </style>
</head>
<body>
  <div class="quiz-container" role="main" aria-labelledby="title">
    <h1 id="title">🎉 Welcome to the Number Quiz! 🎉</h1>
    <h2 id="question">Let's start counting fun together!</h2>

    <div class="options" id="options" role="list"></div>

    <button id="submitButton" aria-label="Start Quiz">Start ▶️</button>
    <button id="nextButton" aria-label="Next question">Next ➡️</button>

    <div class="result" id="result" aria-live="polite"></div>
    <div id="score">Score: 0 ✅</div>
  </div>

  <script>
    const questionDisplay = document.getElementById('question');
    const optionsContainer = document.getElementById('options');
    const resultDisplay = document.getElementById('result');
    const submitButton = document.getElementById('submitButton');
    const nextButton = document.getElementById('nextButton');
    const scoreDisplay = document.getElementById('score');

    let selectedAnswer = null;
    let questionIndex = 0;
    let questions = [];
    let correctCount = 0;
    let quizStarted = false;

    function generateUniqueNumbers(count,min,max){
      const s=new Set();
      while(s.size<count) s.add(Math.floor(Math.random()*(max-min+1))+min);
      return Array.from(s);
    }

    function buildQuestions(){
      questions = [];
      const after = generateUniqueNumbers(20,1,30);
      after.forEach(n => questions.push({type:'after',text:`What comes after ${n}?`,correct:n+1}));
      const before = generateUniqueNumbers(20,2,31);
      before.forEach(n => questions.push({type:'before',text:`What comes before ${n}?`,correct:n-1}));
      const between = generateUniqueNumbers(20,1,28);
      between.forEach(n => questions.push({type:'between',text:`What comes between ${n} and ${n+2}?`,correct:n+1}));
      questionIndex = 0;
    }

    function getFemaleVoice(){
      const synth = window.speechSynthesis;
      const voices = synth.getVoices() || [];
      if(!voices.length) return null;
      const candidates = voices.filter(v => /female|woman|girl/i.test(v.name) || /female|woman|girl/i.test(v.voiceURI));
      if(candidates.length) return candidates[0];
      const en = voices.find(v => v.lang && v.lang.startsWith('en'));
      return en || voices[0];
    }

    function speak(text){
      try{
        const synth = window.speechSynthesis;
        const u = new SpeechSynthesisUtterance(text);
        u.pitch = 1.05; u.rate = 0.95; u.volume = 1;
        const v = getFemaleVoice(); if(v) u.voice = v;
        synth.cancel(); synth.speak(u);
      }catch(e){console.warn('speech error', e)}
    }

    function createOptions(correct){
      optionsContainer.innerHTML = '';
      const set = new Set([correct]);
      while(set.size < 4){
        const cand = correct + Math.floor(Math.random()*5) - 2;
        if(cand > 0 && cand <= 40) set.add(cand);
      }
      const arr = Array.from(set).sort(() => Math.random() - 0.5);
      arr.forEach(n => {
        const b = document.createElement('button');
        b.className = 'option-btn'; b.type = 'button'; b.dataset.value = n; b.textContent = n;
        optionsContainer.appendChild(b);
      });
    }

    optionsContainer.addEventListener('click', e => {
      const btn = e.target.closest('button');
      if(!btn) return;
      const val = Number(btn.dataset.value || btn.textContent.trim());
      selectAnswer(val, btn);
    });

    function selectAnswer(num, btn){
      selectedAnswer = Number(num);
      Array.from(optionsContainer.children).forEach(b => b.classList.remove('selected'));
      btn.classList.add('selected');
      resultDisplay.textContent = '';
    }

    submitButton.addEventListener('click', () => {
      if(!quizStarted){
        quizStarted = true;
        submitButton.textContent = 'Submit ✅';
        buildQuestions();
        newQuestion();
        speak('Let’s begin our number adventure! What comes next?');
        return;
      }

      if(!questions || questions.length === 0){ buildQuestions(); newQuestion(); return; }

      const current = questions[questionIndex];
      if(!current){ buildQuestions(); newQuestion(); return; }

      if(selectedAnswer === null){ resultDisplay.style.color = '#d32f2f'; resultDisplay.textContent = '👆 Pick one!'; speak('Pick one.'); return; }

      submitButton.style.display = 'none'; nextButton.style.display = 'inline-block';

      if(Number(selectedAnswer) === Number(current.correct)){
        correctCount++;
        scoreDisplay.textContent = `Score: ${correctCount} ✅`;
        resultDisplay.style.color = '#388e3c'; resultDisplay.textContent = `🎉 Correct! Great job! You found the right number! Your score is now ${correctCount}!`;
        speak(`Yay! Great job! You found the right number! Your score is now ${correctCount}!`);
      } else {
        if(current.type === 'after'){ resultDisplay.textContent = '❌ Not yet. Think of the next number when you count. Like one, two... what comes next?'; speak('Not yet. Think of the next number when you count. Like one, two... what comes next?'); }
        else if(current.type === 'before'){ resultDisplay.textContent = '❌ Not yet. Think of the number that comes just before when you count back. Like two, one... what is before?'; speak('Not yet. Think of the number that comes just before when you count back. Like two, one... what is before?'); }
        else { resultDisplay.textContent = '❌ Not yet. The number in between sits right in the middle of two friends!'; speak('Not yet. The number in between sits right in the middle of two friends!'); }
        resultDisplay.style.color = '#d32f2f';
      }
    });

    nextButton.addEventListener('click', () => {
      try{ window.speechSynthesis.cancel(); }catch(e){}
      questionIndex++;
      selectedAnswer = null;
      submitButton.style.display = 'inline-block'; nextButton.style.display = 'none';
      newQuestion();
    });

    function newQuestion(){
      resultDisplay.textContent = '';
      if(!questions || questions.length === 0) buildQuestions();
      if(questionIndex >= questions.length){
        questionDisplay.textContent = `🎉 All done! Great job! You got ${correctCount} correct answers! 🎉`;
        optionsContainer.innerHTML = '';
        submitButton.style.display = 'none'; nextButton.style.display = 'none';
        speak(`All done! Great job! You got ${correctCount} correct answers!`);
        return;
      }
      const current = questions[questionIndex];
      questionDisplay.textContent = current.text;
      createOptions(current.correct);
      speak(current.text);
    }

    window.addEventListener('load', () => {
      if(window.speechSynthesis.getVoices().length > 0){
        speak('Welcome to the Number Quiz! Click start when you are ready!');
      } else {
        window.speechSynthesis.onvoiceschanged = () => speak('Welcome to the Number Quiz! Click start when you are ready!');
      }
    });
  </script>
</body>
</html>



