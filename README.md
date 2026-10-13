from pathlib import Path

html = r'''<!DOCTYPE html>
<html lang="vi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Gian Lận Trong Giờ Thi</title>
<style>
  :root { --ink:#18243b; --paper:#fffaf0; --green:#269957; --red:#d94b4b; --gold:#ffcc54; }
  * { box-sizing:border-box }
  body {
    margin:0; min-height:100vh; font-family:system-ui,-apple-system,Segoe UI,sans-serif;
    color:var(--ink); background:linear-gradient(135deg,#d9edff,#f9e6c8);
    display:flex; justify-content:center; align-items:center; padding:20px;
  }
  .game { width:min(900px,100%); background:var(--paper); border:4px solid var(--ink);
    border-radius:22px; overflow:hidden; box-shadow:0 12px 0 #18243b25; }
  header { background:#213653; color:white; text-align:center; padding:24px 18px; }
  h1 { margin:0; font-size:clamp(1.8rem,5vw,3rem); letter-spacing:1px; }
  header p { margin:8px 0 0; color:#dce9ff; }
  .stats { display:grid; grid-template-columns:repeat(3,1fr); gap:10px; padding:14px; background:#e7edf5; }
  .stat { background:white; border:2px solid #d1dbea; border-radius:12px; padding:10px; text-align:center; }
  .stat small { display:block; color:#66738a; font-weight:700; }
  .stat strong { font-size:1.35rem; }
  .meter { height:12px; background:#e3d8d2; border-radius:20px; overflow:hidden; margin-top:7px; }
  #alertBar { height:100%; width:15%; background:#36a66b; transition:width .2s,background .2s; }
  main { padding:clamp(16px,4vw,30px); }
  .scene { display:flex; align-items:center; gap:14px; background:#e7f2ff; border:2px solid #c4d9f3;
    border-radius:16px; padding:14px; margin-bottom:18px; }
  .emoji { font-size:3rem; line-height:1; }
  .scene p { margin:4px 0 0; }
  .question { font-size:1.25rem; font-weight:800; margin:16px 0; }
  .answers { display:grid; grid-template-columns:1fr 1fr; gap:10px; }
  button { cursor:pointer; font:inherit; font-weight:750; border:2px solid var(--ink); border-radius:12px; padding:12px 14px;
    transition:transform .08s,filter .15s; }
  button:hover { filter:brightness(1.04); transform:translateY(-1px); }
  button:disabled { cursor:default; opacity:.65; transform:none; }
  .answer { text-align:left; background:white; }
  .answer:hover { background:#fff2c9; }
  .actions { display:flex; flex-wrap:wrap; gap:10px; margin-top:16px; }
  .primary { background:var(--green); color:white; }
  .cheat { background:#ffcf67; }
  .quiet { background:#e5eaf2; }
  #message { min-height:48px; padding:12px; margin-top:14px; border-radius:12px; background:#f1f3f7; font-weight:650; }
  .hidden { display:none !important; }
  .center { text-align:center; padding:24px 10px; }
  .center h2 { font-size:2rem; }
  footer { background:#213653; color:#dce9ff; text-align:center; padding:12px; font-size:.9rem; }
  @media(max-width:560px) { .answers { grid-template-columns:1fr; } .stats { gap:6px; padding:9px; } .stat { padding:8px 4px; } .scene { align-items:flex-start; } }
</style>
</head>
<body>
<div class="game">
  <header>
    <h1>📝 Gian Lận Trong Giờ Thi</h1>
    <p>Vượt qua bài thi — nhưng coi chừng giám thị!</p>
  </header>
  <div class="stats">
    <div class="stat"><small>ĐIỂM</small><strong id="score">0</strong></div>
    <div class="stat"><small>THỜI GIAN</small><strong><span id="time">60</span>s</strong></div>
    <div class="stat"><small>MỨC NGHI NGỜ</small><strong id="alertText">15%</strong><div class="meter"><div id="alertBar"></div></div></div>
  </div>
  <main>
    <section id="intro" class="center">
      <div style="font-size:4rem">🏫 👨‍🏫 📚</div>
      <h2>Ca thi bắt đầu!</h2>
      <p>Trả lời câu hỏi để ghi điểm. Bạn có thể dùng “liếc bài” để nhận gợi ý, nhưng hành động này làm giám thị nghi ngờ.</p>
      <button class="primary" id="startBtn">BẮT ĐẦU THI ▶</button>
    </section>
    <section id="play" class="hidden">
      <div class="scene">
        <div class="emoji" id="teacherEmoji">👨‍🏫</div>
        <div><strong id="teacherStatus">Giám thị đang nhìn bảng.</strong><p id="sceneTip">Hãy bình tĩnh và chọn đáp án!</p></div>
      </div>
      <div id="progress" style="font-weight:750;color:#65718a">Câu 1 / 8</div>
      <div class="question" id="question">Câu hỏi sẽ xuất hiện ở đây.</div>
      <div class="answers" id="answers"></div>
      <div class="actions">
        <button class="cheat" id="peekBtn">👀 Liếc bài bạn (+gợi ý)</button>
        <button class="quiet" id="calmBtn">😌 Giả vờ suy nghĩ</button>
      </div>
      <div id="message" role="status">Chọn đáp án để bắt đầu.</div>
    </section>
    <section id="result" class="center hidden">
      <div style="font-size:4rem" id="resultEmoji">🏆</div>
      <h2 id="resultTitle">Kết quả</h2>
      <p id="resultText"></p>
      <button class="primary" id="restartBtn">CHƠI LẠI ↻</button>
    </section>
  </main>
  <footer>Game mini chạy trên trình duyệt • Có thể đăng miễn phí bằng GitHub Pages</footer>
</div>
<script>
const questions = [
 {q:"1. 2 + 3 × 2 bằng bao nhiêu?", a:["10","12","8","7"], c:2, hint:"Thực hiện phép nhân trước phép cộng."},
 {q:"2. Hành tinh nào gần Mặt Trời nhất?", a:["Trái Đất","Sao Thủy","Sao Hỏa","Sao Kim"], c:1, hint:"Tên hành tinh có chữ “Thủy”."},
 {q:"3. Thủ đô của Việt Nam là gì?", a:["Huế","Đà Nẵng","Hà Nội","TP. Hồ Chí Minh"], c:2, hint:"Nằm ở miền Bắc."},
 {q:"4. Nước sôi ở khoảng bao nhiêu độ C ở áp suất thường?", a:["50°C","80°C","100°C","120°C"], c:2, hint:"Một trăm độ."},
 {q:"5. Từ nào là danh từ?", a:["Chạy","Đẹp","Bàn","Nhanh"], c:2, hint:"Đồ vật có thể đặt sách lên."},
 {q:"6. Một tuần có bao nhiêu ngày?", a:["5","6","7","8"], c:2, hint:"Từ thứ Hai đến Chủ nhật."},
 {q:"7. Kết quả của 9 × 6 là?", a:["42","54","56","63"], c:1, hint:"50 cộng 4."},
 {q:"8. Khi làm bài thi, cách chắc chắn nhất để đạt điểm tốt là gì?", a:["Học và ôn tập","Chỉ đoán đáp án","Nói chuyện liên tục","Không đọc đề"], c:0, hint:"Chuẩn bị bài trước khi thi!"}
];
let order=[], index=0, score=0, timeLeft=60, suspicion=15, timer=null, usedPeek=false, ended=false;
const $ = id => document.getElementById(id);
function shuffle(arr){ return [...arr].sort(()=>Math.random()-.5); }
function updateStats(){
 $('score').textContent=score; $('time').textContent=timeLeft;
 $('alertText').textContent=suspicion+'%'; $('alertBar').style.width=suspicion+'%';
 $('alertBar').style.background=suspicion>=75?'#d94b4b':suspicion>=45?'#f0a52b':'#36a66b';
}
function startGame(){
 clearInterval(timer); order=shuffle(questions); index=0; score=0; timeLeft=60; suspicion=15; usedPeek=false; ended=false;
 $('intro').classList.add('hidden'); $('result').classList.add('hidden'); $('play').classList.remove('hidden');
 updateStats(); renderQuestion();
 timer=setInterval(()=>{ if(ended)return; timeLeft--; updateStats(); if(timeLeft<=0) finishGame(); },1000);
}
function renderQuestion(){
 if(index>=order.length){finishGame();return;}
 const item=order[index]; $('progress').textContent=`Câu ${index+1} / ${order.length}`;
 $('question').textContent=item.q; $('answers').innerHTML='';
 item.a.forEach((answer,i)=>{
   const b=document.createElement('button'); b.className='answer'; b.textContent=String.fromCharCode(65+i)+'. '+answer;
   b.onclick=()=>chooseAnswer(i,b); $('answers').appendChild(b);
 });
 $('message').textContent='Chọn đáp án đúng để ghi điểm.'; $('peekBtn').disabled=false;
 $('teacherEmoji').textContent=suspicion>=75?'👀':'👨‍🏫';
 $('teacherStatus').textContent=suspicion>=75?'Giám thị đang nghi ngờ bạn!':'Giám thị đang quan sát lớp.';
 $('sceneTip').textContent=suspicion>=75?'Cẩn thận! Mức nghi ngờ đã cao.':'Bình tĩnh đọc kỹ câu hỏi.';
}
function chooseAnswer(choice,btn){
 if(ended)return;
 const item=order[index]; [...$('answers').children].forEach(b=>b.disabled=true);
 if(choice===item.c){ score+=10; $('message').textContent='✅ Chính xác! +10 điểm.'; btn.style.background='#d8f8e4'; }
 else { suspicion=Math.min(100,suspicion+8); $('message').textContent='❌ Chưa đúng! Mức nghi ngờ tăng nhẹ.'; btn.style.background='#ffe0e0'; }
 updateStats();
 if(suspicion>=100){ setTimeout(()=>finishGame(true),650); return; }
 index++; setTimeout(renderQuestion,650);
}
$('peekBtn').onclick=()=>{
 if(ended||usedPeek)return;
 usedPeek=true; suspicion=Math.min(100,suspicion+22); updateStats();
 $('message').textContent='🗒️ Gợi ý: '+order[index].hint+' (Giám thị bắt đầu để ý!)';
 $('peekBtn').disabled=true; $('teacherEmoji').textContent='👀';
 $('teacherStatus').textContent='Giám thị vừa quay xuống!';
 if(suspicion>=100)finishGame(true);
};
$('calmBtn').onclick=()=>{
 if(ended)return;
 suspicion=Math.max(0,suspicion-10); timeLeft=Math.max(0,timeLeft-3); updateStats();
 $('message').textContent='😌 Bạn giả vờ suy nghĩ rất tự nhiên. Giảm 10% nghi ngờ nhưng mất 3 giây!';
 $('teacherEmoji').textContent='🤔'; $('teacherStatus').textContent='Giám thị quay lên bảng.';
};
function finishGame(caught=false){
 if(ended)return; ended=true; clearInterval(timer);
 $('play').classList.add('hidden'); $('result').classList.remove('hidden');
 const title=caught?'BỊ BẮT QUẢ TANG!':timeLeft<=0?'HẾT GIỜ!':'HOÀN THÀNH BÀI THI!';
 $('resultTitle').textContent=title;
 $('resultEmoji').textContent=caught?'🚨':score>=60?'🏆':'📚';
 $('resultText').textContent=`Bạn đạt ${score} điểm, còn ${timeLeft} giây và mức nghi ngờ ${suspicion}%. ${caught?'Giám thị đã phát hiện hành vi đáng ngờ!':score>=60?'Làm tốt lắm! Lần sau thử đạt điểm cao hơn.':'Hãy luyện tập thêm và thử lại nhé!'}`;
 updateStats();
}
$('startBtn').onclick=startGame; $('restartBtn').onclick=startGame;
</script>
</body>
</html>'''

path = Path("/mnt/data/gian-lan-trong-gio-thi.html")
path.write_text(html, encoding="utf-8")
print(f"Đã tạo game: {path}")
