<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>AI Planner</title>
  <style>
    :root {
      --sogang-red: #B01030;
      --sogang-gray: #58595B;
      --bg: #F2F2F2;
      --white: #FFFFFF;
    }

    body {
      font-family: 'Pretendard', sans-serif;
      background-color: var(--bg);
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      margin: 0;
      color: var(--sogang-gray);
    }

    .container {
      width: 100%;
      max-width: 700px;
      background: var(--white);
      padding: 70px 50px;
      box-shadow: 0 10px 40px rgba(0,0,0,0.1);
      border-top: 12px solid var(--sogang-red);
    }

    .header { text-align: center; margin-bottom: 60px; }
    .header h1 { font-size: 42px; font-weight: 800; margin: 0; color: var(--sogang-red); }
    .header .sub-text { font-size: 18px; margin-top: 10px; text-transform: uppercase; letter-spacing: 1px; }

    .progress-bar { width: 100%; height: 4px; background: #E1E1E1; margin: 40px 0 10px; }
    .progress-fill { width: 33.3%; height: 100%; background: var(--sogang-red); transition: width 0.4s; }
    .step-label { text-align: right; font-size: 16px; color: var(--sogang-red); font-weight: 600; }

    .step { display: none; }
    .step.active { display: block; animation: fadeIn 0.5s; }

    @keyframes fadeIn {
      from { opacity: 0; transform: translateY(10px); }
      to { opacity: 1; transform: translateY(0); }
    }

    h2 { font-size: 32px; text-align: center; margin-bottom: 50px; font-weight: 700; }

    .btn-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; }
    .opt-btn {
      padding: 35px 20px; border: 1px solid #E1E1E1; background: var(--white);
      cursor: pointer; font-size: 24px; transition: all 0.2s; font-weight: 500;
      display: flex; justify-content: center; align-items: center;
    }
    .opt-btn.selected { border-color: var(--sogang-red); background: var(--sogang-red); color: var(--white); box-shadow: 0 4px 15px rgba(176,16,48,0.2); }

    .nav-btns { margin-top: 60px; display: flex; gap: 15px; }
    .prev-btn { flex: 1; background: #E1E1E1; border: none; padding: 22px; font-size: 20px; font-weight: 600; cursor: pointer; color: var(--sogang-gray); }
    .next-btn { flex: 2; background: var(--sogang-red); color: var(--white); border: none; padding: 22px; font-size: 20px; font-weight: 600; cursor: pointer; }

    #result-content { background: #fdfdfd; padding: 40px; border-radius: 10px; border-left: 8px solid var(--sogang-red); line-height: 1.6; }
    .analysis-item { margin-bottom: 20px; font-size: 20px; }
    .recommendation { background: #fff5f6; padding: 25px; border-radius: 10px; color: var(--sogang-red); font-weight: bold; font-size: 22px; }

    .loader { border: 5px solid #f3f3f3; border-top: 5px solid var(--sogang-red); border-radius: 50%; width: 50px; height: 50px; animation: spin 1s linear infinite; margin: 0 auto 20px; }
    @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
  </style>
</head>
<body>

<div class="container">
  <div class="header">
    <h1><span style="font-family:'Times New Roman'; font-style:italic;">AI</span> PLANNER</h1>
    <div class="sub-text">Personal Management System</div>
    <div class="progress-bar"><div class="progress-fill" id="fill"></div></div>
    <div class="step-label" id="step-text">STEP 01 / 03</div>
  </div>

  <div class="step active" id="step1">
    <h2>어떤 종류의 목표인가요?</h2>
    <div class="btn-grid">
      <button class="opt-btn" onclick="select('type', '시험', this)">📚 시험 (Exam)</button>
      <button class="opt-btn" onclick="select('type', '과제', this)">📝 과제 (Assignment)</button>
      <button class="opt-btn" onclick="select('type', '자격증', this)">🏆 자격증 (Cert.)</button>
      <button class="opt-btn" onclick="select('type', '자기계발', this)">🌱 자기계발 (Self)</button>
      <button class="opt-btn" onclick="select('type', '운동', this)">🏃 운동 (Workout)</button>
      <button class="opt-btn" onclick="select('type', '기타', this)">🔍 기타 (Etc.)</button>
    </div>
  </div>

  <div class="step" id="step2">
    <h2>현재 스트레스 지수는?</h2>
    <div class="btn-grid">
      <button class="opt-btn" onclick="select('stress', '낮음', this)">😊 낮음 (Low)</button>
      <button class="opt-btn" onclick="select('stress', '보통', this)">😐 보통 (Medium)</button>
      <button class="opt-btn" onclick="select('stress', '높음', this)">😫 높음 (High)</button>
      <button class="opt-btn" onclick="select('stress', '매우높음', this)">🆘 매우높음 (Max)</button>
    </div>
  </div>

  <div class="step" id="step3">
    <h2>오늘의 계획 달성률은?</h2>
    <div class="btn-grid">
      <button class="opt-btn" onclick="select('done', '100', this)">🔥 100%</button>
      <button class="opt-btn" onclick="select('done', '50', this)">🌗 50%</button>
      <button class="opt-btn" onclick="select('done', '20', this)">🌑 20%</button>
      <button class="opt-btn" onclick="select('done', '0', this)">🚫 0%</button>
    </div>
  </div>

  <div class="step" id="step-loading">
    <div style="text-align: center; padding: 50px 0;">
      <div class="loader"></div>
      <p style="font-size: 24px; font-weight: 600;">AI가 오늘의 기록을 분석 중입니다...</p>
    </div>
  </div>

  <div class="step" id="step-result">
    <h2>데이터 분석 결과</h2>
    <div id="result-content">
      <div class="analysis-item" id="analysis-summary"></div>
      <div class="recommendation" id="ai-recom"></div>
    </div>
    <button class="next-btn" style="margin-top: 40px; width: 100%;" onclick="location.reload()">다시 시작하기</button>
  </div>

  <div class="nav-btns" id="nav-area">
    <button class="prev-btn" onclick="moveStep(-1)">PREV</button>
    <button class="next-btn" onclick="moveStep(1)">NEXT STEP</button>
  </div>
</div>

<script>
  let currentStep = 1;
  let userData = { type: '', stress: '', done: '' };

  function select(key, val, btn) {
    userData[key] = val;
    const btns = btn.parentElement.querySelectorAll('.opt-btn');
    btns.forEach(b => b.classList.remove('selected'));
    btn.classList.add('selected');
    
    // 편리함을 위해 선택하면 바로 다음 버튼으로 시선이 가게 함 (선택 효과 강조)
  }

  function moveStep(n) {
    // 다음으로 넘어갈 때 선택을 안 했으면 경고 (사용자 편의성)
    if (n === 1) {
      if (currentStep === 1 && !userData.type) return alert("목표를 선택해주세요!");
      if (currentStep === 2 && !userData.stress) return alert("스트레스 지수를 선택해주세요!");
      if (currentStep === 3 && !userData.done) return alert("달성률을 선택해주세요!");
    }

    if (n === 1 && currentStep < 3) {
      currentStep++;
      updateUI();
    } else if (n === -1 && currentStep > 1) {
      currentStep--;
      updateUI();
    } else if (n === 1 && currentStep === 3) {
      showResult();
    }
  }

  function updateUI() {
    // 모든 스텝 숨기기
    const steps = document.querySelectorAll('.step');
    steps.forEach(s => s.classList.remove('active'));
    
    // 현재 스텝 표시
    document.getElementById('step' + currentStep).classList.add('active');
    
    // 프로그레스 바 업데이트
    const fillPercent = (currentStep / 3) * 100;
    document.getElementById('fill').style.width = fillPercent + '%';
    document.getElementById('step-text').innerText = 'STEP 0' + currentStep + ' / 03';
  }

  function showResult() {
    document.querySelectorAll('.step').forEach(s => s.classList.remove('active'));
    document.getElementById('step-loading').classList.add('active');
    document.getElementById('nav-area').style.display = 'none';

    setTimeout(() => {
      document.getElementById('step-loading').classList.remove('active');
      document.getElementById('step-result').classList.add('active');

      let advice = "";
      const isHighStress = (userData.stress === '높음' || userData.stress === '매우높음');
      const doneRate = parseInt(userData.done);

      if (doneRate === 100) {
        advice = "완벽한 하루였습니다! 현재의 리듬을 유지하되, 내일은 15분 정도의 고난도 심화 과업을 추가해봐도 좋습니다. [생산성 최상]";
      } 
      else if (doneRate >= 50) {
        advice = isHighStress 
          ? "절반 이상 달성했지만 스트레스가 높네요. 내일 오전은 가벼운 복습 위주로 배치하여 심리적 압박을 줄이는 것을 추천합니다. [회복 우선]"
          : "무난한 하루였지만 조금 더 집중할 수 있습니다. 내일은 가장 에너지가 좋은 시간에 '핵심 목표'를 먼저 처리해보세요. [집중력 보완]";
      } 
      else if (doneRate >= 20) {
        advice = isHighStress 
          ? "오늘 많이 힘드셨군요. 낮은 달성률은 휴식이 부족하다는 신호입니다. 내일 계획의 50%를 삭제하고 재충전 시간을 가지세요. [강제 휴식 권고]"
          : "컨디션은 괜찮지만 실행력이 부족했습니다. 내일은 목표를 더 작게 쪼개서 '작은 성공'을 맛보는 것부터 시작합시다. [실행력 강화]";
      } 
      else {
        advice = "오늘의 기록이 내일의 밑거름이 될 거예요. 자책하기보다 내일 아침 10분간 '딱 하나'만 하겠다는 마음으로 다시 시작해봅시다. [리마인드]";
      }

      document.getElementById('analysis-summary').innerHTML = 
        `오늘 <strong>${userData.type}</strong> 목표를 향해 달렸으며, <strong>${userData.done}%</strong>의 달성률을 보였습니다.`;
      document.getElementById('ai-recom').innerHTML = `💡 AI 맞춤 제언: ${advice}`;
    }, 1500);
  }
</script>

</body>
</html>
