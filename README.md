## skypulse-weather App
A modern, responsive weather application that provides real-time weather updates for any city worldwide. This project fetches data from the OpenWeatherMap API to display current temperature, humidity, wind speed, and weather conditions (like "Clear Sky" or "Rain").
<img width="771" height="763" alt="Screenshot 2026-04-17 152050" src="https://github.com/user-attachments/assets/1ba21ef4-31a3-4e6b-a29e-9fe615b6df8f" />

Weather App
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8"/>
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>SkyPulse — Weather App</title>
  <link href="https://fonts.googleapis.com/css2?family=Syne:wght@400;600;700;800&family=DM+Sans:wght@300;400;500&display=swap" rel="stylesheet"/>
  <style>
    *,*::before,*::after{box-sizing:border-box;margin:0;padding:0}
    :root{
      --sky:#0a1628;--mid:#112240;
      --card:rgba(255,255,255,0.06);--border:rgba(255,255,255,0.1);
      --accent:#ccd6da;--accent2:#818cf8;--warm:#fb923c;	
      --text:#e2e8f0;--muted:#94a3b8;--radius:20px;
      --red:rgb(118, 5, 12);--red:#a71414;
    }
    body{font-family:'DM Sans',sans-serif;background:var(--sky);color:var(--text);min-height:100vh;overflow-x:hidden}
    .bg-canvas{
      position:fixed;inset:0;z-index:0;
      background:radial-gradient(ellipse at 20% 20%,#1e3a5f 0%,transparent 50%),
                 radial-gradient(ellipse at 80% 80%,#3a16bd 0%,transparent 50%),
                 linear-gradient(135deg,#1a4b95 0%,#164d95 100%);
    }
    .stars{position:fixed;inset:0;z-index:0;pointer-events:none}
    .star{position:absolute;background:white;border-radius:50%;animation:twinkle var(--dur) ease-in-out infinite}
    @keyframes twinkle{0%,100%{opacity:0.1;transform:scale(1)}50%{opacity:0.9;transform:scale(1.3)}}
    .container{position:relative;z-index:1;max-width:920px;margin:0 auto;padding:2rem 1.5rem 4rem}

    /* HEADER */
    header{text-align:center;margin-bottom:2.5rem;animation:fadeDown 0.6s ease both}
    .logo{font-family:'Syne',sans-serif;font-size:1rem;font-weight:800;letter-spacing:.3em;text-transform:uppercase;color:var(--accent);margin-bottom:.25rem}
    h1{font-family:'Syne',sans-serif;font-size:clamp(2rem,5vw,3.2rem);font-weight:800;background:linear-gradient(135deg,#e2e8f0 0%,var(--accent) 100%);-webkit-background-clip:text;-webkit-text-fill-color:transparent;line-height:1.1}
    header p{color:var(--muted);font-size:.95rem;margin-top:.5rem}

    /* API STATUS BAR */
    .api-bar{display:flex;align-items:center;justify-content:center;gap:.5rem;padding:.5rem 1rem;border-radius:999px;background:var(--card);border:1px solid var(--border);font-size:.75rem;color:var(--muted);margin-bottom:1.5rem;backdrop-filter:blur(10px)}
    .api-dot{width:8px;height:8px;border-radius:50%;background:var(--green);animation:pulse 2s ease-in-out infinite}
    .api-dot.err{background:var(--red);animation:none}
    @keyframes pulse{0%,100%{opacity:1}50%{opacity:.4}}

    /* SEARCH */
    .search-wrapper{display:flex;gap:.75rem;margin-bottom:1rem;animation:fadeUp 0.6s .1s ease both}
    .search-wrapper input{flex:1;background:var(--card);border:1px solid var(--border);border-radius:var(--radius);padding:1rem 1.4rem;font-family:'DM Sans',sans-serif;font-size:1rem;color:var(--text);outline:none;transition:border-color .2s,box-shadow .2s;backdrop-filter:blur(10px)}
    .search-wrapper input::placeholder{color:var(--muted)}
    .search-wrapper input:focus{border-color:var(--accent);box-shadow:0 0 0 3px rgba(74, 188, 237, 0.15)}
    .btn{background:linear-gradient(135deg,var(--accent),var(--accent2));border:none;border-radius:var(--radius);padding:1rem 1.8rem;font-family:'Syne',sans-serif;font-weight:700;font-size:.9rem;color:white;cursor:pointer;letter-spacing:.05em;transition:transform .15s,box-shadow .15s;white-space:nowrap}
    .btn:hover{transform:translateY(-2px);box-shadow:0 8px 24px rgba(56,189,248,.3)}
    .btn:active{transform:translateY(0)}
    .btn-outline{background:transparent;border:1px solid var(--border);border-radius:var(--radius);padding:.75rem 1.2rem;color:var(--muted);cursor:pointer;font-size:1rem;transition:all .2s;backdrop-filter:blur(10px)}
    .btn-outline:hover{border-color:var(--accent);color:var(--accent)}

    /* QUICK CHIPS */
    .chips-row{display:flex;flex-wrap:wrap;gap:.4rem;margin-bottom:1.5rem;animation:fadeUp 0.5s .2s ease both}
    .chip{background:var(--card);border:1px solid var(--border);border-radius:999px;padding:.35rem 1rem;font-size:.78rem;color:var(--muted);cursor:pointer;transition:all .2s;backdrop-filter:blur(10px)}
    .chip:hover{border-color:var(--accent);color:var(--accent);transform:translateY(-1px)}

    /* STATUS */
    .status-msg{text-align:center;padding:1.5rem;border-radius:var(--radius);background:var(--card);border:1px solid var(--border);font-size:.95rem;backdrop-filter:blur(10px);animation:fadeUp .3s ease both;margin-bottom:1rem}
    .status-msg.error{color:#8a070715;border-color:rgba(143, 6, 6, 0.3)}
    .status-msg.loading{color:var(--muted)}
    .spinner{display:inline-block;width:18px;height:18px;border:2px solid var(--border);border-top-color:var(--accent);border-radius:50%;animation:spin .7s linear infinite;margin-right:8px;vertical-align:middle}
    @keyframes spin{to{transform:rotate(360deg)}}

    /* MAIN CARD */
    .weather-main{background:var(--card);border:1px solid var(--border);border-radius:28px;padding:2.2rem;backdrop-filter:blur(20px);margin-bottom:1.5rem;animation:fadeUp .5s ease both;position:relative;overflow:hidden}
    .weather-main::before{content:'';position:absolute;top:-60px;right:-60px;width:200px;height:200px;background:radial-gradient(circle,rgba(56,189,248,.12) 0%,transparent 70%);pointer-events:none}
    .location-row{display:flex;align-items:center;justify-content:space-between;flex-wrap:wrap;gap:.5rem;margin-bottom:1.5rem}
    .city-name{font-family:'Syne',sans-serif;font-size:clamp(1.5rem,4vw,2.2rem);font-weight:800}
    .local-time{font-size:.85rem;color:var(--muted);margin-top:.1rem}
    .weather-badge{background:rgba(56,189,248,.12);border:1px solid rgba(56,189,248,.2);border-radius:50px;padding:.4rem 1rem;font-size:.8rem;color:var(--accent);font-weight:500}
    .temp-row{display:flex;align-items:center;gap:1.5rem;flex-wrap:wrap}
    .temp-big{font-family:'Syne',sans-serif;font-size:clamp(4rem,12vw,7rem);font-weight:800;line-height:1;background:linear-gradient(135deg,#ffffff 0%,var(--accent) 100%);-webkit-background-clip:text;-webkit-text-fill-color:transparent}
    .weather-icon-big{font-size:clamp(4rem,10vw,6rem);filter:drop-shadow(0 0 20px rgba(56,189,248,.4));animation:float 4s ease-in-out infinite}
    @keyframes float{0%,100%{transform:translateY(0)}50%{transform:translateY(-10px)}}
    .condition-text{font-size:1.1rem;color:var(--muted);margin-top:.3rem;text-transform:capitalize}
    .feels-like{font-size:.85rem;color:var(--muted);margin-top:.15rem}
    .temp-toggle{display:flex;gap:.4rem;margin-top:.8rem}
    .toggle-btn{background:transparent;border:1px solid var(--border);border-radius:8px;padding:.3rem .8rem;color:var(--muted);font-family:'Syne',sans-serif;font-size:.8rem;font-weight:700;cursor:pointer;transition:all .2s}
    .toggle-btn.active{background:var(--accent);border-color:var(--accent);color:rgb(131, 3, 3)}

    /* STATS */
    .stats-grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(130px,1fr));gap:1rem;margin-bottom:1.5rem;animation:fadeUp .5s .1s ease both}
    .stat-card{background:var(--card);border:1px solid var(--border);border-radius:var(--radius);padding:1.2rem;backdrop-filter:blur(10px);transition:transform .2s,border-color .2s}
    .stat-card:hover{transform:translateY(-3px);border-color:rgba(56,189,248,.3)}
    .stat-icon{font-size:1.5rem;margin-bottom:.5rem}
    .stat-label{font-size:.75rem;color:var(--muted);text-transform:uppercase;letter-spacing:.06em;margin-bottom:.3rem}
    .stat-value{font-family:'Syne',sans-serif;font-size:1.25rem;font-weight:700}
    .stat-sub{font-size:.75rem;color:var(--muted);margin-top:.2rem}

    /* HOURLY */
    .section-box{background:var(--card);border:1px solid var(--border);border-radius:var(--radius);padding:1.4rem;backdrop-filter:blur(10px);margin-bottom:1.5rem;animation:fadeUp .5s .2s ease both}
    .section-title{font-family:'Syne',sans-serif;font-size:.8rem;font-weight:700;letter-spacing:.12em;text-transform:uppercase;color:var(--muted);margin-bottom:1rem}
    .hourly-scroll{display:flex;gap:.75rem;overflow-x:auto;padding-bottom:.5rem;scrollbar-width:none}
    .hourly-scroll::-webkit-scrollbar{display:none}
    .hour-card{flex:0 0 70px;background:rgba(255,255,255,.04);border:1px solid var(--border);border-radius:14px;padding:.9rem .5rem;text-align:center;transition:background .2s,border-color .2s}
    .hour-card.now{background:rgba(56,189,248,.12);border-color:rgba(56,189,248,.3)}
    .hour-card:hover{background:rgba(255,255,255,.08)}
    .hour-time{font-size:.7rem;color:var(--muted);margin-bottom:.4rem}
    .hour-icon{font-size:1.4rem;margin-bottom:.4rem}
    .hour-temp{font-family:'Syne',sans-serif;font-size:.9rem;font-weight:700}
    .hour-pop{font-size:.65rem;color:var(--accent);margin-top:.2rem}

    /* FORECAST */
    .forecast-row{display:flex;align-items:center;padding:.75rem 0;border-bottom:1px solid var(--border);gap:1rem}
    .forecast-row:last-child{border-bottom:none}
    .forecast-day{font-family:'Syne',sans-serif;font-weight:700;font-size:.9rem;min-width:50px}
    .forecast-icon{font-size:1.4rem;flex:0 0 32px}
    .forecast-desc{font-size:.8rem;color:var(--muted);flex:1;text-transform:capitalize}
    .forecast-pop{font-size:.75rem;color:var(--accent);min-width:36px;text-align:right}
    .forecast-temps{display:flex;gap:.75rem;font-family:'Syne',sans-serif;font-weight:700;font-size:.9rem}
    .temp-high{color:var(--warm)}.temp-low{color:var(--muted);font-weight:400}

    /* SUN */
    .sun-times{display:flex;justify-content:space-around;margin-top:.75rem}
    .sun-item{text-align:center}
    .sun-label{font-size:.7rem;color:var(--muted);text-transform:uppercase;letter-spacing:.08em}
    .sun-val{font-family:'Syne',sans-serif;font-size:1rem;font-weight:700;color:var(--warm);margin-top:.2rem}
    .day-length{text-align:center;font-size:.8rem;color:var(--muted);margin-top:.6rem}

    /* EMPTY */
    .empty-state{text-align:center;padding:4rem 2rem;animation:fadeUp .6s ease both}
    .empty-state .big-icon{font-size:5rem;margin-bottom:1rem}
    .empty-state h2{font-family:'Syne',sans-serif;font-size:1.5rem;font-weight:700;margin-bottom:.5rem}
    .empty-state p{color:var(--muted);font-size:.95rem;max-width:400px;margin:0 auto}

    @keyframes fadeDown{from{opacity:0;transform:translateY(-20px)}to{opacity:1;transform:none}}
    @keyframes fadeUp{from{opacity:0;transform:translateY(20px)}to{opacity:1;transform:none}}
    @media(max-width:520px){.stats-grid{grid-template-columns:repeat(2,1fr)}.search-wrapper{flex-wrap:wrap}.search-wrapper .btn{flex:1}}
  </style>
</head>
<body>
<div class="bg-canvas"></div>
<div class="stars" id="stars"></div>

<div class="container">
  <header>
    <div class="logo">☁ SkyPulse</div>
    <h1>Weather App</h1>
    <p>Real-time forecasts · No API key · Free forever</p>
  </header>

  <div class="api-bar" id="apiBar">
    <div class="api-dot" id="apiDot"></div>
    <span id="apiLabel">Checking API connection…</span>
  </div>

  <div class="search-wrapper">
    <input type="text" id="cityInput" placeholder="City, state or country — e.g. Mumbai, California, France…" autocomplete="off"
      onkeydown="if(event.key==='Enter')searchWeather()"/>
    <button class="btn" onclick="searchWeather()">Search</button>
    <button class="btn-outline" onclick="geoMe()" title="Use my location">📍</button>
  </div>

  <div class="chips-row" id="chips"></div>
  <div id="statusMsg"></div>
  <div id="weatherOutput"></div>
</div>

<script>
/* ── State ── */
let isCel = true, last = null;

/* ── Stars ── */
const se = document.getElementById('stars');
for(let i=0;i<80;i++){
  const s=document.createElement('div');s.className='star';
  const z=Math.random()*2+1;
  s.style.cssText=`width:${z}px;height:${z}px;top:${Math.random()*100}%;left:${Math.random()*100}%;--dur:${2+Math.random()*4}s;animation-delay:${Math.random()*3}s`;
  se.appendChild(s);
}

/* ── Quick chips ── */
const CITIES=['Delhi','Mumbai','Bangalore','Chennai','Kolkata','New York','London','Tokyo','Paris','Sydney','Dubai','Singapore','Berlin','Toronto','Seoul'];
const cr=document.getElementById('chips');
CITIES.forEach(c=>{
  const b=document.createElement('button');b.className='chip';b.textContent=c;
  b.onclick=()=>{document.getElementById('cityInput').value=c;searchWeather()};
  cr.appendChild(b);
});

/* ── WMO codes ── */
const WMO={
  0:{l:'Clear sky',e:d=>d?'☀️':'🌙'},
  1:{l:'Mainly clear',e:d=>d?'🌤️':'🌙'},
  2:{l:'Partly cloudy',e:()=>'⛅'},
  3:{l:'Overcast',e:()=>'☁️'},
  45:{l:'Foggy',e:()=>'🌫️'},48:{l:'Icy fog',e:()=>'🌫️'},
  51:{l:'Light drizzle',e:()=>'🌦️'},53:{l:'Drizzle',e:()=>'🌦️'},55:{l:'Heavy drizzle',e:()=>'🌧️'},
  61:{l:'Light rain',e:()=>'🌧️'},63:{l:'Moderate rain',e:()=>'🌧️'},65:{l:'Heavy rain',e:()=>'🌧️'},
  71:{l:'Light snow',e:()=>'❄️'},73:{l:'Snow',e:()=>'❄️'},75:{l:'Heavy snow',e:()=>'🌨️'},77:{l:'Snow grains',e:()=>'🌨️'},
  80:{l:'Light showers',e:()=>'🌦️'},81:{l:'Showers',e:()=>'🌧️'},82:{l:'Violent showers',e:()=>'⛈️'},
  85:{l:'Snow showers',e:()=>'🌨️'},86:{l:'Heavy snow showers',e:()=>'🌨️'},
  95:{l:'Thunderstorm',e:()=>'⛈️'},96:{l:'Storm+hail',e:()=>'⛈️'},99:{l:'Storm+hail',e:()=>'⛈️'}
};
function wi(c,d){const w=WMO[c]||{l:'Unknown',e:()=>'🌡️'};return{label:w.l,emoji:w.e(d)}}

/* ── Helpers ── */
function showMsg(m,t='loading'){
  document.getElementById('statusMsg').innerHTML=`<div class="status-msg ${t}">${m}</div>`;
  document.getElementById('weatherOutput').innerHTML='';
}
function clearMsg(){document.getElementById('statusMsg').innerHTML=''}
function toF(c){return c*9/5+32}
function fmt(c){return isCel?`${Math.round(c)}°C`:`${Math.round(toF(c))}°F`}
function fH(iso){return new Date(iso).toLocaleTimeString('en-US',{hour:'numeric',hour12:true})}
function fT(iso){return new Date(iso).toLocaleTimeString('en-US',{hour:'2-digit',minute:'2-digit'})}
function fDay(iso){return new Date(iso).toLocaleDateString('en-US',{weekday:'short'})}
function wDir(d){return['N','NE','E','SE','S','SW','W','NW'][Math.round(d/45)%8]}
function uvL(u){if(u==null)return'—';if(u<3)return'Low';if(u<6)return'Moderate';if(u<8)return'High';if(u<11)return'Very High';return'Extreme'}
function flag(cc){if(!cc||cc.length!==2)return'';return[...cc.toUpperCase()].map(c=>String.fromCodePoint(0x1F1E6-65+c.charCodeAt(0))).join('')}

/* ── API ping check ── */
async function checkAPI(){
  try{
    const r=await fetch('https://api.open-meteo.com/v1/forecast?latitude=28.6&longitude=77.2&current=temperature_2m&timezone=auto',{signal:AbortSignal.timeout(5000)});
    const d=await r.json();
    if(d.current){
      document.getElementById('apiDot').className='api-dot';
      document.getElementById('apiLabel').textContent='Open-Meteo API connected · Free · No key needed';
    } else throw new Error();
  }catch{
    document.getElementById('apiDot').className='api-dot err';
    document.getElementById('apiLabel').textContent='API unreachable — check internet connection';
  }
}
checkAPI();

/* ── Geocoding (Open-Meteo, free) ── */
async function searchWeather(){
  const q=document.getElementById('cityInput').value.trim();
  if(!q)return;
  showMsg('<span class="spinner"></span> Searching for "'+q+'"…');
  try{
    const r=await fetch(
      `https://geocoding-api.open-meteo.com/v1/search?name=${encodeURIComponent(q)}&count=5&language=en&format=json`,
      {signal:AbortSignal.timeout(8000)}
    );
    if(!r.ok)throw new Error(`HTTP ${r.status}`);
    const d=await r.json();
    if(!d.results||!d.results.length){
      showMsg('❌  "'+q+'" not found. Try adding country — e.g. "Paris, France" or "Springfield, Illinois"','error');
      return;
    }
    const loc=d.results[0];
    loadWeather(loc.latitude,loc.longitude,loc.name,loc.country_code||'',loc.country||'',loc.admin1||'',loc.timezone||'auto');
  }catch(e){
    showMsg('❌ Network error: '+e.message+'. Check your internet connection.','error');
  }
}

/* ── GPS ── */
async function geoMe(){
  if(!navigator.geolocation){showMsg('❌ Geolocation not supported by your browser.','error');return}
  showMsg('<span class="spinner"></span> Getting your location…');
  navigator.geolocation.getCurrentPosition(async pos=>{
    const{latitude:la,longitude:lo}=pos.coords;
    try{
      const r=await fetch(`https://nominatim.openstreetmap.org/reverse?lat=${la}&lon=${lo}&format=json&accept-language=en`,{signal:AbortSignal.timeout(6000)});
      const d=await r.json();
      const name=d.address?.city||d.address?.town||d.address?.village||d.address?.county||'Your Location';
      const cc=(d.address?.country_code||'').toUpperCase();
      const country=d.address?.country||'';
      const state=d.address?.state||'';
      loadWeather(la,lo,name,cc,country,state,Intl.DateTimeFormat().resolvedOptions().timeZone);
    }catch{
      loadWeather(la,lo,'Your Location','','','','auto');
    }
  },err=>{
    const msgs={1:'Location access denied.',2:'Position unavailable.',3:'Location request timed out.'};
    showMsg('❌ '+(msgs[err.code]||'Unknown error.'),'error');
  },{timeout:10000});
}

/* ── Weather fetch ── */
async function loadWeather(lat,lon,name,cc,country,state,tz){
  showMsg('<span class="spinner"></span> Loading weather for '+name+'…');
  const params=new URLSearchParams({
    latitude:lat,longitude:lon,
    current:'temperature_2m,relative_humidity_2m,apparent_temperature,is_day,weather_code,cloud_cover,wind_speed_10m,wind_direction_10m,surface_pressure,visibility',
    hourly:'temperature_2m,weather_code,precipitation_probability',
    daily:'weather_code,temperature_2m_max,temperature_2m_min,precipitation_probability_max,sunrise,sunset,uv_index_max',
    wind_speed_unit:'kmh',
    timezone:tz||'auto',
    forecast_days:7
  });
  try{
    const r=await fetch(`https://api.open-meteo.com/v1/forecast?${params}`,{signal:AbortSignal.timeout(10000)});
    if(!r.ok)throw new Error(`HTTP ${r.status}`);
    const d=await r.json();
    if(d.error)throw new Error(d.reason||'API error');
    last={d,name,cc,country,state};
    clearMsg();
    render(last);
  }catch(e){
    showMsg(`❌ Failed: ${e.message}. Try again or check your connection.`,'error');
  }
}

/* ── Render ── */
function render({d,name,cc,country,state}){
  const cur=d.current,hr=d.hourly,day=d.daily;
  const isDay=cur.is_day===1;
  const{label,emoji}=wi(cur.weather_code,isDay);
  const now=new Date().toLocaleTimeString('en-US',{hour:'2-digit',minute:'2-digit'});
  const locFull=[name,state,country].filter(Boolean).join(', ');

  let hi=hr.time.findIndex(t=>t>=cur.time);if(hi<0)hi=0;
  const hrs=Array.from({length:12},(_,i)=>({
    time:hr.time[hi+i]||'',
    temp:hr.temperature_2m[hi+i]??0,
    wc:hr.weather_code[hi+i]??0,
    pop:hr.precipitation_probability[hi+i]??0
  }));

  const sr=day.sunrise[0],ss=day.sunset[0];
  const srMs=new Date(sr).getTime(),ssMs=new Date(ss).getTime();
  const dLen=Math.max(ssMs-srMs,1);
  const pct=Math.max(0,Math.min(1,(Date.now()-srMs)/dLen));
  const sunX=20+pct*360, sunY=Math.max(5,80-Math.sin(pct*Math.PI)*95);

  document.getElementById('weatherOutput').innerHTML=`
    <div class="weather-main">
      <div class="location-row">
        <div>
          <div class="city-name">${name} ${flag(cc)}</div>
          <div class="local-time">${[state,country].filter(Boolean).join(', ')} · ${now}</div>
        </div>
        <div class="weather-badge">${label}</div>
      </div>
      <div class="temp-row">
        <div class="weather-icon-big">${emoji}</div>
        <div>
          <div class="temp-big" id="tb">${fmt(cur.temperature_2m)}</div>
          <div class="condition-text">${label}</div>
          <div class="feels-like">Feels like ${fmt(cur.apparent_temperature)} · H: ${fmt(day.temperature_2m_max[0])} · L: ${fmt(day.temperature_2m_min[0])}</div>
          <div class="temp-toggle">
            <button class="toggle-btn ${isCel?'active':''}" onclick="setU(true)">°C</button>
            <button class="toggle-btn ${!isCel?'active':''}" onclick="setU(false)">°F</button>
          </div>
        </div>
      </div>
    </div>

    <div class="stats-grid">
      <div class="stat-card"><div class="stat-icon">💧</div><div class="stat-label">Humidity</div><div class="stat-value">${cur.relative_humidity_2m}%</div><div class="stat-sub">${cur.relative_humidity_2m<40?'Dry':cur.relative_humidity_2m<70?'Comfortable':'Humid'}</div></div>
      <div class="stat-card"><div class="stat-icon">💨</div><div class="stat-label">Wind</div><div class="stat-value">${(cur.wind_speed_10m??0).toFixed(1)} km/h</div><div class="stat-sub">${wDir(cur.wind_direction_10m??0)}</div></div>
      <div class="stat-card"><div class="stat-icon">👁️</div><div class="stat-label">Visibility</div><div class="stat-value">${cur.visibility!=null?(cur.visibility/1000).toFixed(1):'—'} km</div><div class="stat-sub">${(cur.visibility??0)>=10000?'Clear':(cur.visibility??0)>=5000?'Moderate':'Low'}</div></div>
      <div class="stat-card"><div class="stat-icon">🌡️</div><div class="stat-label">Pressure</div><div class="stat-value">${Math.round(cur.surface_pressure??1013)} hPa</div><div class="stat-sub">${(cur.surface_pressure??1013)<1000?'Low':(cur.surface_pressure??1013)>1020?'High':'Normal'}</div></div>
      <div class="stat-card"><div class="stat-icon">☁️</div><div class="stat-label">Cloud Cover</div><div class="stat-value">${cur.cloud_cover??0}%</div><div class="stat-sub">${(cur.cloud_cover??0)<25?'Clear':(cur.cloud_cover??0)<50?'Partly cloudy':(cur.cloud_cover??0)<75?'Mostly cloudy':'Overcast'}</div></div>
      <div class="stat-card"><div class="stat-icon">🔆</div><div class="stat-label">UV Index</div><div class="stat-value">${day.uv_index_max[0]?.toFixed(1)??'—'}</div><div class="stat-sub">${uvL(day.uv_index_max[0])}</div></div>
    </div>

    <div class="section-box">
      <div class="section-title">⏱ Hourly Forecast</div>
      <div class="hourly-scroll">
        ${hrs.map((h,i)=>{const{emoji:e}=wi(h.wc,true);return`<div class="hour-card ${i===0?'now':''}"><div class="hour-time">${i===0?'Now':fH(h.time)}</div><div class="hour-icon">${e}</div><div class="hour-temp">${fmt(h.temp)}</div>${h.pop>10?`<div class="hour-pop">💧${Math.round(h.pop)}%</div>`:''}</div>`}).join('')}
      </div>
    </div>

    <div class="section-box">
      <div class="section-title">📅 7-Day Forecast</div>
      ${day.time.map((t,i)=>{
        const{label:dl,emoji:de}=wi(day.weather_code[i],true);
        const pop=day.precipitation_probability_max[i]??0;
        return`<div class="forecast-row">
          <div class="forecast-day">${i===0?'Today':fDay(t)}</div>
          <div class="forecast-icon">${de}</div>
          <div class="forecast-desc">${dl}</div>
          ${pop>10?`<div class="forecast-pop">💧${pop}%</div>`:'<div class="forecast-pop"></div>'}
          <div class="forecast-temps"><span class="temp-high">${fmt(day.temperature_2m_max[i])}</span><span class="temp-low">${fmt(day.temperature_2m_min[i])}</span></div>
        </div>`;
      }).join('')}
    </div>

    <div class="section-box">
      <div class="section-title">🌅 Sun & Daylight</div>
      <svg viewBox="0 0 400 90" style="width:100%;height:80px" xmlns="http://www.w3.org/2000/svg">
        <defs>
          <linearGradient id="ag" x1="0%" y1="0%" x2="100%" y2="0%">
            <stop offset="0%" stop-color="#fb923c" stop-opacity="0.3"/>
            <stop offset="50%" stop-color="#facc15" stop-opacity="0.6"/>
            <stop offset="100%" stop-color="#f97316" stop-opacity="0.3"/>
          </linearGradient>
          <filter id="gl"><feGaussianBlur stdDeviation="4" result="b"/><feMerge><feMergeNode in="b"/><feMergeNode in="SourceGraphic"/></feMerge></filter>
        </defs>
        <path d="M 20 80 Q 200 -20 380 80" fill="none" stroke="url(#ag)" stroke-width="2" stroke-dasharray="4 4"/>
        <circle cx="${sunX}" cy="${sunY}" r="10" fill="#facc15" filter="url(#gl)"/>
      </svg>
      <div class="sun-times">
        <div class="sun-item"><div class="sun-label">🌅 Sunrise</div><div class="sun-val">${fT(sr)}</div></div>
        <div class="sun-item"><div class="sun-label">🌇 Sunset</div><div class="sun-val">${fT(ss)}</div></div>
      </div>
      <div class="day-length">Day length: ${Math.floor(dLen/3600000)}h ${Math.floor((dLen%3600000)/60000)}m</div>
    </div>
  `;
}

function setU(c){if(isCel===c)return;isCel=c;if(last)render(last)}

/* ── Empty state ── */
document.getElementById('weatherOutput').innerHTML=`
  <div class="empty-state">
    <div class="big-icon">🌍</div>
    <h2>Explore the Weather</h2>
    <p>Search any city, state or country — or tap 📍 to use your current location.</p>
  </div>
`;
</script>
</body>
</html>S

