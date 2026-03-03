<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>AbdullahTok</title>

<style>
body{
  margin:0;
  background:black;
  color:white;
  font-family:Tahome;
  overflow:hidden;
}

video{
  width:100%;
  height:100vh;
  object-fit:cover;
}

.screen{display:none}
.active{display:block}

.sideButtons{
  position:absolute;
  right:10px;
  bottom:120px;
  text-align:center;
  font-size:20px;
}

.sideButtons div{
  margin-bottom:25px;
  cursor:pointer;
}

.bottomInfo{
  position:absolute;
  bottom:80px;
  left:15px;
}

.avatar{
  width:45px;
  height:45px;
  border-radius:50%;
}

.topBar{
  position:fixed;
  top:0;
  width:100%;
  background:#111;
  padding:10px;
  display:flex;
  justify-content:space-between;
  z-index:1000;
}

input,button{
  padding:6px;
  border:none;
  border-radius:8px;
}

button{
  background:#00ff88;
  font-weight:bold;
  cursor:pointer;
}
</style>
</head>
<body>

<div class="topBar">
  <strong>AbdullahTok</strong>
  <input id="search" placeholder="بحث" onkeyup="searchVideo()">
</div>

<div id="feed" class="screen active"></div>

<div id="profile" class="screen" style="padding:20px;margin-top:50px">
  <h2>الملف الشخصي</h2>
  <input id="name" placeholder="اسمك">
  <input type="file" accept="image/*" onchange="setAvatar(event)">
  <button onclick="saveProfile()">حفظ</button>

  <h3 id="profileName"></h3>
  <img id="profileAvatar" class="avatar">

  <p>المتابعين: <span id="followers">0</span></p>
  <p>المتابَعين: <span id="following">0</span></p>
  <p>إجمالي اللايكات: <span id="likesCount">0</span></p>
  <p>إجمالي المشاهدات: <span id="viewsCount">0</span></p>

  <input type="file" accept="video/*" onchange="uploadVideo(event)">
</div>

<div style="position:fixed;bottom:0;width:100%;display:flex;justify-content:space-around;background:#111;padding:10px">
  <button onclick="showFeed()">الرئيسية</button>
  <button onclick="showProfile()">حسابي</button>
</div>

<script>

let videos = JSON.parse(localStorage.getItem("videos")||"[]");
let profile = JSON.parse(localStorage.getItem("profile")||"{}");
let selectedAvatar=null;

function saveAll(){
  localStorage.setItem("videos",JSON.stringify(videos));
  localStorage.setItem("profile",JSON.stringify(profile));
}

function showFeed(){
  document.getElementById("feed").classList.add("active");
  document.getElementById("profile").classList.remove("active");
  renderFeed();
}

function showProfile(){
  document.getElementById("profile").classList.add("active");
  document.getElementById("feed").classList.remove("active");
  renderProfile();
}

function setAvatar(e){
  let reader=new FileReader();
  reader.onload=function(ev){
    selectedAvatar=ev.target.result;
  }
  reader.readAsDataURL(e.target.files[0]);
}

function saveProfile(){
  profile.name=document.getElementById("name").value;
  if(selectedAvatar) profile.avatar=selectedAvatar;
  profile.followers=profile.followers||0;
  profile.following=profile.following||0;
  saveAll();
  renderProfile();
}

function renderProfile(){
  document.getElementById("profileName").innerText=profile.name||"";
  document.getElementById("profileAvatar").src=profile.avatar||"";
  document.getElementById("followers").innerText=profile.followers||0;
  document.getElementById("following").innerText=profile.following||0;

  let likes=0,views=0;
  videos.forEach(v=>{
    if(v.owner===profile.name){
      likes+=v.likes;
      views+=v.views;
    }
  });

  document.getElementById("likesCount").innerText=likes;
  document.getElementById("viewsCount").innerText=views;
}

function uploadVideo(e){
  let reader=new FileReader();
  reader.onload=function(ev){
    let vid={
      id:Date.now(),
      src:ev.target.result,
      owner:profile.name,
      avatar:profile.avatar,
      title:"فيديو جديد",
      likes:0,
      views:0,
      comments:[]
    };
    videos.unshift(vid);
    saveAll();
    showFeed();
  }
  reader.readAsDataURL(e.target.files[0]);
}

function renderFeed(){
  let feed=document.getElementById("feed");
  feed.innerHTML="";
  videos.forEach(v=>{
    let div=document.createElement("div");
    div.style.position="relative";
    div.innerHTML=`
      <video src="${v.src}" onclick="addView(${v.id})" autoplay loop controls></video>

      <div class="bottomInfo">
        <img src="${v.avatar}" class="avatar">
        <div>${v.title}</div>
      </div>

      <div class="sideButtons">
        <div onclick="likeVideo(${v.id})">❤️ ${v.likes}</div>
        <div onclick="commentVideo(${v.id})">💬 ${v.comments.length}</div>
        <div onclick="shareVideo()">🔗</div>
      </div>
    `;
    feed.appendChild(div);
  });
}

function likeVideo(id){
  let v=videos.find(x=>x.id===id);
  v.likes++;
  saveAll();
  renderFeed();
}

function addView(id){
  let v=videos.find(x=>x.id===id);
  v.views++;
  saveAll();
}

function commentVideo(id){
  let text=prompt("اكتب تعليق");
  if(text){
    let v=videos.find(x=>x.id===id);
    v.comments.push(text);
    saveAll();
    renderFeed();
  }
}

function shareVideo(){
  alert("تم نسخ الرابط (تجريبي)");
}

function searchVideo(){
  let value=document.getElementById("search").value.toLowerCase();
  let filtered=videos.filter(v=>v.title.toLowerCase().includes(value));
  let feed=document.getElementById("feed");
  feed.innerHTML="";
  filtered.forEach(v=>{
    let div=document.createElement("div");
    div.innerHTML=`<video src="${v.src}" controls></video>`;
    feed.appendChild(div);
  });
}

window.onload=renderFeed;

</script>

</body>
</html>
