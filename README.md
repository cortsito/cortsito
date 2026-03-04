import { useState, useEffect, useRef } from "react";

// ── CONSTANTS ─────────────────────────────────────────────
const EM = "#10b981";
const EM2 = "#34d399";
const EM3 = "#6ee7b7";
const BG = "#040906";
const SURF = "#080f0b";
const BORDER = "rgba(16,185,129,0.18)";
const TEXT = "#d1fae5";
const GHOST = "rgba(209,250,229,0.55)";
const DIM = "rgba(209,250,229,0.25)";
const FONT = "'JetBrains Mono', 'Courier New', monospace";

// ── GRID BACKGROUND ───────────────────────────────────────
const GridBg = () => {
  const ref = useRef(null);
  useEffect(() => {
    const c = ref.current; if (!c) return;
    const resize = () => { c.width = window.innerWidth; c.height = window.innerHeight; };
    resize();
    const ctx = c.getContext("2d");
    const step = 48;
    ctx.clearRect(0,0,c.width,c.height);
    ctx.strokeStyle = "rgba(16,185,129,0.055)";
    ctx.lineWidth = 0.5;
    for (let x = 0; x < c.width; x += step) {
      ctx.beginPath(); ctx.moveTo(x,0); ctx.lineTo(x,c.height); ctx.stroke();
    }
    for (let y = 0; y < c.height; y += step) {
      ctx.beginPath(); ctx.moveTo(0,y); ctx.lineTo(c.width,y); ctx.stroke();
    }
    // dots at intersections
    ctx.fillStyle = "rgba(16,185,129,0.12)";
    for (let x = 0; x < c.width; x += step)
      for (let y = 0; y < c.height; y += step) {
        ctx.beginPath(); ctx.arc(x,y,1,0,Math.PI*2); ctx.fill();
      }
    window.addEventListener("resize", resize);
    return () => window.removeEventListener("resize", resize);
  },[]);
  return <canvas ref={ref} style={{position:"fixed",inset:0,pointerEvents:"none",opacity:1}}/>;
};

// ── SCANLINE ──────────────────────────────────────────────
const Scanline = () => {
  const ref = useRef(null);
  useEffect(() => {
    const el = ref.current; if (!el) return;
    let y = -10, af;
    const tick = () => {
      y = (y + 0.4) % (window.innerHeight + 20);
      el.style.top = y + "px";
      af = requestAnimationFrame(tick);
    };
    tick(); return () => cancelAnimationFrame(af);
  },[]);
  return <div ref={ref} style={{
    position:"fixed",left:0,right:0,height:"2px",pointerEvents:"none",zIndex:9999,
    background:"linear-gradient(to bottom,transparent,rgba(16,185,129,0.06),transparent)",
  }}/>;
};

// ── BOOT SEQUENCE ─────────────────────────────────────────
const BOOT_LINES = [
  "[ SYS ] Iniciando sistema de identificación...",
  "[ OK  ] Kernel de ingeniería cargado",
  "[ OK  ] Módulo Full-Stack — activo",
  "[ OK  ] Módulo AI/ML/Data — activo",
  "[ OK  ] Módulo Blockchain/Web3 — activo",
  "[ OK  ] Módulo Infraestructura — activo",
  "[ WARN] Neo-polímata en proceso — dominio incompleto (intencional)",
  "[ OK  ] Cargando perfil: cortsito",
  "[ SYS ] Listo.",
];

const Boot = ({ onDone }) => {
  const [lines, setLines] = useState([]);
  const [done, setDone] = useState(false);
  useEffect(() => {
    let i = 0;
    const interval = setInterval(() => {
      if (i < BOOT_LINES.length) { setLines(l => [...l, BOOT_LINES[i]]); i++; }
      else { clearInterval(interval); setTimeout(() => { setDone(true); setTimeout(onDone, 400); }, 300); }
    }, 160);
    return () => clearInterval(interval);
  }, []);
  return (
    <div style={{
      position:"fixed",inset:0,background:BG,display:"flex",alignItems:"center",
      justifyContent:"center",zIndex:100,
      opacity: done ? 0 : 1, transition:"opacity 0.4s",pointerEvents: done?"none":"all",
    }}>
      <GridBg/>
      <div style={{fontFamily:FONT,fontSize:"0.75rem",color:EM,lineHeight:2,
        maxWidth:"500px",padding:"2rem"}}>
        {lines.map((l,i) => (
          <div key={i} style={{
            opacity: i === lines.length-1 ? 1 : 0.6,
            color: l.includes("WARN") ? "#fbbf24" : l.includes("SYS") ? EM2 : EM,
            animation:"fadeIn 0.2s ease",
          }}>{l}</div>
        ))}
        {!done && <span style={{color:EM,animation:"blink 1s infinite"}}>█</span>}
      </div>
      <style>{`
        @keyframes blink { 0%,100%{opacity:1} 50%{opacity:0} }
        @keyframes fadeIn { from{opacity:0;transform:translateY(4px)} to{opacity:1;transform:none} }
      `}</style>
    </div>
  );
};

// ── SKILL BAR ─────────────────────────────────────────────
const Bar = ({ label, sub, level, delay = 0 }) => {
  const [w, setW] = useState(0);
  useEffect(() => { const t = setTimeout(() => setW(level), delay + 200); return () => clearTimeout(t); }, []);
  return (
    <div style={{marginBottom:"0.85rem"}}>
      <div style={{display:"flex",justifyContent:"space-between",marginBottom:"4px"}}>
        <span style={{fontFamily:FONT,fontSize:"0.72rem",color:TEXT,letterSpacing:"0.04em"}}>{label}</span>
        <span style={{fontFamily:FONT,fontSize:"0.6rem",color:DIM}}>{sub}</span>
      </div>
      <div style={{height:"2px",background:"rgba(16,185,129,0.1)",borderRadius:"1px",overflow:"hidden"}}>
        <div style={{
          height:"100%",width:`${w}%`,background:`linear-gradient(to right,${EM},${EM2})`,
          transition:"width 1.2s cubic-bezier(0.4,0,0.2,1)",borderRadius:"1px",
        }}/>
      </div>
    </div>
  );
};

// ── TAG ───────────────────────────────────────────────────
const Tag = ({ children, dim }) => (
  <span style={{
    display:"inline-block",padding:"3px 10px",fontFamily:FONT,fontSize:"0.63rem",
    border:`1px solid ${dim ? "rgba(16,185,129,0.12)" : BORDER}`,
    color: dim ? DIM : EM2,
    background: dim ? "transparent" : "rgba(16,185,129,0.05)",
    borderRadius:"2px",marginRight:"6px",marginBottom:"6px",letterSpacing:"0.06em",
  }}>{children}</span>
);

// ── SECTION HEADER ─────────────────────────────────────────
const SH = ({ num, title }) => (
  <div style={{marginBottom:"1.5rem",display:"flex",alignItems:"center",gap:"0.75rem"}}>
    <span style={{fontFamily:FONT,fontSize:"0.58rem",color:DIM,letterSpacing:"0.2em"}}>{num}</span>
    <div style={{flex:1,height:"1px",background:BORDER}}/>
    <span style={{fontFamily:FONT,fontSize:"0.65rem",color:EM,letterSpacing:"0.18em"}}>{title}</span>
    <div style={{width:"8px",height:"8px",border:`1px solid ${EM}`,transform:"rotate(45deg)",flexShrink:0}}/>
  </div>
);

// ── MODULE CARD ───────────────────────────────────────────
const Card = ({ id, title, desc, children, accent }) => (
  <div style={{
    background:SURF,border:`1px solid ${BORDER}`,borderRadius:"2px",
    padding:"1.5rem",position:"relative",overflow:"hidden",
  }}>
    <div style={{position:"absolute",top:0,left:0,width:"3px",height:"100%",background:accent||EM}}/>
    <div style={{display:"flex",gap:"0.75rem",alignItems:"baseline",marginBottom:"0.5rem"}}>
      <span style={{fontFamily:FONT,fontSize:"0.55rem",color:DIM}}>{id}</span>
      <span style={{fontFamily:FONT,fontSize:"0.8rem",color:TEXT,letterSpacing:"0.08em"}}>{title}</span>
    </div>
    {desc && <p style={{fontFamily:FONT,fontSize:"0.68rem",color:GHOST,lineHeight:1.75,
      marginBottom:"1rem",letterSpacing:"0.02em"}}>{desc}</p>}
    {children}
  </div>
);

// ── MAIN ──────────────────────────────────────────────────
export default function App() {
  const [booted, setBooted] = useState(false);
  const [tab, setTab] = useState("identidad");
  const [visible, setVisible] = useState(false);

  useEffect(() => {
    if (booted) setTimeout(() => setVisible(true), 100);
  }, [booted]);

  const tabs = [
    { id:"identidad",    label:"01 · identidad"    },
    { id:"fullstack",    label:"02 · full-stack"    },
    { id:"ai",           label:"03 · ai · datos"    },
    { id:"web3",         label:"04 · blockchain"    },
    { id:"infra",        label:"05 · infraestructura"},
    { id:"filosofia",    label:"06 · filosofía"     },
  ];

  return (
    <div style={{background:BG,minHeight:"100vh",color:TEXT}}>
      <Boot onDone={() => setBooted(true)} />
      <GridBg/>
      <Scanline/>

      <div style={{
        maxWidth:"900px",margin:"0 auto",padding:"2.5rem 1.5rem",
        position:"relative",
        opacity: visible ? 1 : 0,
        transform: visible ? "none" : "translateY(12px)",
        transition:"opacity 0.6s ease, transform 0.6s ease",
      }}>

        {/* ── HEADER ── */}
        <div style={{marginBottom:"3rem"}}>
          <div style={{
            display:"flex",alignItems:"center",gap:"1rem",marginBottom:"1.5rem",
            fontFamily:FONT,fontSize:"0.6rem",color:DIM,letterSpacing:"0.2em",
          }}>
            <div style={{width:"8px",height:"8px",background:EM,animation:"pulse 2s infinite"}}/>
            SISTEMA ACTIVO · PERFIL DE INGENIERÍA · github.com/cortsito
          </div>

          <div style={{position:"relative",marginBottom:"1rem"}}>
            <div style={{fontFamily:FONT,fontSize:"0.6rem",color:DIM,letterSpacing:"0.2em",marginBottom:"0.5rem"}}>
              IDENTIFICACIÓN ·············
            </div>
            <div style={{
              fontSize:"clamp(2.2rem, 6vw, 4rem)",fontFamily:FONT,fontWeight:700,
              letterSpacing:"-0.02em",lineHeight:1,
              color:"transparent",
              WebkitTextStroke:`1px ${EM}`,
              marginBottom:"0.25rem",
            }}>
              cortsito
            </div>
            <div style={{
              fontFamily:FONT,fontSize:"clamp(0.7rem, 2vw, 0.85rem)",
              color:GHOST,letterSpacing:"0.12em",
            }}>
              / full-stack · ai/ml · blockchain · infra / neo-polímata
            </div>
          </div>

          <div style={{
            fontFamily:FONT,fontSize:"0.7rem",color:GHOST,lineHeight:1.9,
            maxWidth:"600px",borderLeft:`2px solid ${BORDER}`,paddingLeft:"1rem",
            letterSpacing:"0.03em",
          }}>
            Constructor de sistemas en la intersección de producto, datos e infraestructura.<br/>
            Aprendo en serio: 20% de los conceptos que desbloquean el 80% del sistema.<br/>
            <span style={{color:DIM}}>// Teamlead técnico · Arquitecto · Siempre construyendo.</span>
          </div>
        </div>

        {/* ── NAV TABS ── */}
        <div style={{
          display:"flex",flexWrap:"wrap",gap:"0",
          borderBottom:`1px solid ${BORDER}`,marginBottom:"2.5rem",
        }}>
          {tabs.map(t => (
            <button key={t.id} onClick={() => setTab(t.id)} style={{
              padding:"0.6rem 1rem",fontFamily:FONT,fontSize:"0.62rem",
              letterSpacing:"0.1em",border:"none",cursor:"pointer",background:"none",
              color: tab===t.id ? EM2 : DIM,
              borderBottom: tab===t.id ? `2px solid ${EM}` : "2px solid transparent",
              transition:"all 0.2s",
            }}>{t.label}</button>
          ))}
        </div>

        {/* ── IDENTIDAD ── */}
        {tab==="identidad" && (
          <div style={{animation:"fadeIn 0.3s ease"}}>
            <SH num="01" title="DIAGNÓSTICO GENERAL"/>

            <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:"1rem",marginBottom:"1.5rem"}}>
              {[
                ["ROL ACTUAL","Arquitecto técnico / Team lead"],
                ["PROYECTO ACTIVO","CibusChain — Food rescue infra"],
                ["TRACK","McDonald's @ Feed the Future Hackathon"],
                ["OBJETIVO","Dominio senior en múltiples dominios"],
                ["MÉTODO","Primeros principios · no acumulación"],
                ["ESTADO","Construyendo"],
              ].map(([k,v]) => (
                <div key={k} style={{padding:"0.85rem",background:SURF,border:`1px solid ${BORDER}`,borderRadius:"2px"}}>
                  <div style={{fontFamily:FONT,fontSize:"0.55rem",color:DIM,letterSpacing:"0.18em",marginBottom:"4px"}}>{k}</div>
                  <div style={{fontFamily:FONT,fontSize:"0.73rem",color:TEXT}}>{v}</div>
                </div>
              ))}
            </div>

            <SH num="01.2" title="DOMINIOS ACTIVOS"/>
            <div style={{marginBottom:"2rem"}}>
              {[
                { label:"Full-Stack Engineering",    sub:"React · Next.js · FastAPI · Django",    level:85, delay:0   },
                { label:"AI / ML / Data",            sub:"PyTorch · TensorFlow · LangChain",      level:78, delay:120 },
                { label:"Blockchain / Web3",         sub:"Solidity · EVM · Hardhat · ethers.js",  level:72, delay:240 },
                { label:"Infraestructura / DevOps",  sub:"Docker · AWS · GCP · CI/CD",            level:70, delay:360 },
                { label:"Arquitectura de sistemas",  sub:"Diseño · APIs · Escalabilidad",         level:80, delay:480 },
              ].map(s => <Bar key={s.label} {...s}/>)}
            </div>

            <SH num="01.3" title="GITHUB STATS"/>
            <div style={{
              display:"grid",gridTemplateColumns:"1fr 1fr",gap:"1rem",
              background:SURF,border:`1px solid ${BORDER}`,borderRadius:"2px",padding:"1.25rem",
            }}>
              <div style={{textAlign:"center"}}>
                <img
                  src="https://github-readme-stats.vercel.app/api?username=cortsito&show_icons=true&theme=transparent&hide_border=true&title_color=10b981&text_color=6ee7b7&icon_color=34d399&bg_color=080f0b&ring_color=10b981&include_all_commits=true&count_private=true"
                  alt="GitHub Stats"
                  style={{width:"100%",maxWidth:"380px",borderRadius:"2px"}}
                  onError={e => { e.target.style.display="none"; e.target.nextSibling.style.display="block"; }}
                />
                <div style={{display:"none",fontFamily:FONT,fontSize:"0.65rem",color:DIM,padding:"1rem"}}>
                  [ stats · github.com/cortsito ]
                </div>
              </div>
              <div style={{textAlign:"center"}}>
                <img
                  src="https://github-readme-stats.vercel.app/api/top-langs/?username=cortsito&layout=compact&theme=transparent&hide_border=true&title_color=10b981&text_color=6ee7b7&bg_color=080f0b&langs_count=6"
                  alt="Top Languages"
                  style={{width:"100%",maxWidth:"380px",borderRadius:"2px"}}
                  onError={e => { e.target.style.display="none"; e.target.nextSibling.style.display="block"; }}
                />
                <div style={{display:"none",fontFamily:FONT,fontSize:"0.65rem",color:DIM,padding:"1rem"}}>
                  [ top langs · github.com/cortsito ]
                </div>
              </div>
            </div>
          </div>
        )}

        {/* ── FULL-STACK ── */}
        {tab==="fullstack" && (
          <div style={{animation:"fadeIn 0.3s ease"}}>
            <SH num="02" title="MÓDULO FULL-STACK"/>
            <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:"1rem",marginBottom:"1.5rem"}}>
              <Card id="02.A" title="FRONTEND" accent={EM}>
                {[
                  {l:"React",    s:"hooks · context · patterns"},
                  {l:"Next.js",  s:"SSR · SSG · App Router"},
                  {l:"TypeScript",s:"strict mode · generics"},
                  {l:"Tailwind", s:"utility-first CSS"},
                  {l:"Vite",     s:"build tooling"},
                ].map(({l,s},i) => <Bar key={l} label={l} sub={s} level={88-i*4} delay={i*80}/>)}
              </Card>
              <Card id="02.B" title="BACKEND" accent={EM2}>
                {[
                  {l:"Python",   s:"core language"},
                  {l:"FastAPI",  s:"async · Pydantic · OpenAPI"},
                  {l:"Django",   s:"ORM · DRF · Admin"},
                  {l:"Node.js",  s:"Express · runtime"},
                  {l:"GraphQL",  s:"Apollo · schema design"},
                ].map(({l,s},i) => <Bar key={l} label={l} sub={s} level={87-i*3} delay={i*80}/>)}
              </Card>
            </div>
            <Card id="02.C" title="BASES DE DATOS" accent={EM3}>
              <div style={{display:"flex",flexWrap:"wrap",marginTop:"0.25rem"}}>
                {["PostgreSQL","MongoDB","Redis","SQLite","Supabase","Prisma ORM","SQLAlchemy","Firebase"].map(t=><Tag key={t}>{t}</Tag>)}
              </div>
            </Card>
          </div>
        )}

        {/* ── AI / DATA ── */}
        {tab==="ai" && (
          <div style={{animation:"fadeIn 0.3s ease"}}>
            <SH num="03" title="MÓDULO AI · DATOS · ML"/>
            <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:"1rem",marginBottom:"1.5rem"}}>
              <Card id="03.A" title="FRAMEWORKS ML" accent={EM} desc="Entrenamiento, inferencia y pipelines de modelos.">
                {[
                  {l:"PyTorch",    s:"modelo · backprop · cuda"},
                  {l:"TensorFlow",s:"keras · serving"},
                  {l:"scikit-learn",s:"classical ML"},
                  {l:"Hugging Face",s:"transformers · hub"},
                  {l:"LangChain",  s:"LLM pipelines"},
                ].map(({l,s},i)=><Bar key={l} label={l} sub={s} level={83-i*4} delay={i*80}/>)}
              </Card>
              <Card id="03.B" title="DATOS & INGENIERÍA" accent={EM2} desc="Procesamiento, análisis y pipelines de datos a escala.">
                {[
                  {l:"Pandas",   s:"transformación · análisis"},
                  {l:"NumPy",    s:"álgebra lineal"},
                  {l:"Apache Spark",s:"big data · distribución"},
                  {l:"dbt",      s:"data transformation"},
                  {l:"Airflow",  s:"orquestación"},
                ].map(({l,s},i)=><Bar key={l} label={l} sub={s} level={78-i*3} delay={i*80}/>)}
              </Card>
            </div>
            <Card id="03.C" title="ECOSISTEMA COMPLETO" accent={EM3}>
              <div style={{display:"flex",flexWrap:"wrap",marginTop:"0.25rem"}}>
                {["MLflow","Weights & Biases","FAISS","Chroma","OpenAI API","Anthropic API",
                  "Jupyter","Matplotlib","Seaborn","Plotly","Streamlit","Gradio"].map(t=><Tag key={t}>{t}</Tag>)}
              </div>
            </Card>
          </div>
        )}

        {/* ── BLOCKCHAIN ── */}
        {tab==="web3" && (
          <div style={{animation:"fadeIn 0.3s ease"}}>
            <SH num="04" title="MÓDULO BLOCKCHAIN · WEB3"/>
            <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:"1rem",marginBottom:"1.5rem"}}>
              <Card id="04.A" title="SMART CONTRACTS" accent={EM} desc="Desarrollo y despliegue de contratos en EVM.">
                {[
                  {l:"Solidity",  s:"contratos · patterns · seguridad"},
                  {l:"Hardhat",   s:"testing · deployment"},
                  {l:"ethers.js", s:"interacción frontend-chain"},
                  {l:"OpenZeppelin",s:"estándares · ERC-20/721"},
                  {l:"Foundry",   s:"testing avanzado"},
                ].map(({l,s},i)=><Bar key={l} label={l} sub={s} level={80-i*4} delay={i*80}/>)}
              </Card>
              <Card id="04.B" title="INFRAESTRUCTURA WEB3" accent={EM2} desc="Capas de almacenamiento y oráculos.">
                {[
                  {l:"IPFS",       s:"almacenamiento descentralizado"},
                  {l:"Chainlink",  s:"oráculos · VRF · automation"},
                  {l:"The Graph",  s:"indexing · queries"},
                  {l:"Alchemy",    s:"node provider · APIs"},
                  {l:"Wagmi",      s:"React hooks Web3"},
                ].map(({l,s},i)=><Bar key={l} label={l} sub={s} level={75-i*3} delay={i*80}/>)}
              </Card>
            </div>
            <Card id="04.C" title="CADENAS & PROTOCOLOS" accent={EM3}>
              <div style={{display:"flex",flexWrap:"wrap",marginTop:"0.25rem"}}>
                {["Ethereum","Polygon","Arbitrum","Base","Avalanche","EVM compatible",
                  "ERC-20","ERC-721","ERC-1155","Multisig","DAO patterns"].map(t=><Tag key={t}>{t}</Tag>)}
              </div>
            </Card>
          </div>
        )}

        {/* ── INFRA ── */}
        {tab==="infra" && (
          <div style={{animation:"fadeIn 0.3s ease"}}>
            <SH num="05" title="MÓDULO INFRAESTRUCTURA · DEVOPS"/>
            <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:"1rem",marginBottom:"1.5rem"}}>
              <Card id="05.A" title="CONTENEDORES & CI/CD" accent={EM} desc="Build, test, deploy. Automático.">
                {[
                  {l:"Docker",        s:"compose · multi-stage builds"},
                  {l:"GitHub Actions",s:"CI/CD pipelines"},
                  {l:"Kubernetes",    s:"orquestación básica"},
                  {l:"Nginx",         s:"reverse proxy · config"},
                  {l:"Terraform",     s:"IaC · cloud provisioning"},
                ].map(({l,s},i)=><Bar key={l} label={l} sub={s} level={78-i*4} delay={i*80}/>)}
              </Card>
              <Card id="05.B" title="CLOUD" accent={EM2} desc="Servicios gestionados y serverless.">
                {[
                  {l:"AWS",  s:"EC2 · S3 · Lambda · RDS · ECS"},
                  {l:"GCP",  s:"Cloud Run · BigQuery · GKE"},
                  {l:"Vercel",s:"frontend · edge functions"},
                  {l:"Railway",s:"backend deploys rápidos"},
                  {l:"Cloudflare",s:"CDN · Workers · DNS"},
                ].map(({l,s},i)=><Bar key={l} label={l} sub={s} level={76-i*3} delay={i*80}/>)}
              </Card>
            </div>
            <Card id="05.C" title="OBSERVABILIDAD & MONITOREO" accent={EM3}>
              <div style={{display:"flex",flexWrap:"wrap",marginTop:"0.25rem"}}>
                {["Prometheus","Grafana","Sentry","Datadog","ELK Stack","New Relic","PagerDuty","Loki"].map(t=><Tag key={t}>{t}</Tag>)}
              </div>
            </Card>
          </div>
        )}

        {/* ── FILOSOFÍA ── */}
        {tab==="filosofia" && (
          <div style={{animation:"fadeIn 0.3s ease"}}>
            <SH num="06" title="SISTEMA OPERATIVO MENTAL"/>

            <div style={{
              background:SURF,border:`1px solid ${BORDER}`,borderRadius:"2px",
              padding:"1.5rem",marginBottom:"1.25rem",fontFamily:FONT,
            }}>
              <div style={{fontSize:"0.58rem",color:DIM,letterSpacing:"0.2em",marginBottom:"1rem"}}>
                PRINCIPIOS DE OPERACIÓN ···
              </div>
              {[
                ["ENTENDER > ACUMULAR",     "No colecciono herramientas. Entiendo los sistemas que las generan."],
                ["VELOCIDAD CON RAÍZ",      "Rápido no significa superficial. La velocidad real viene de la comprensión profunda."],
                ["SENIOR O NO CUENTA",      "El estándar es alto o es ruido. Prefiero avanzar lento en lo correcto."],
                ["DOMINIO CRUZADO",         "La intersección entre disciplinas es donde aparecen las soluciones que nadie más ve."],
                ["CONSTRUCCIÓN PRIMERO",    "Los proyectos reales enseñan más que cualquier curso. Build first."],
              ].map(([k,v],i) => (
                <div key={k} style={{
                  borderLeft:`2px solid ${BORDER}`,paddingLeft:"1rem",marginBottom:"1rem",
                  paddingBottom:"1rem",
                  borderBottom: i < 4 ? `1px solid ${BORDER}` : "none",
                }}>
                  <div style={{fontSize:"0.63rem",color:EM,letterSpacing:"0.15em",marginBottom:"4px"}}>{k}</div>
                  <div style={{fontSize:"0.72rem",color:GHOST,lineHeight:1.7,letterSpacing:"0.03em"}}>{v}</div>
                </div>
              ))}
            </div>

            <Card id="06.B" title="PSEUDOCÓDIGO" accent={EM}>
              <pre style={{fontFamily:FONT,fontSize:"0.68rem",color:GHOST,lineHeight:1.9,
                margin:0,whiteSpace:"pre-wrap",letterSpacing:"0.02em"}}>
{`class Cortsito:
    dominios = [
        "full-stack", "ai/ml", "blockchain",
        "infraestructura", "arquitectura"
    ]
    método   = "primeros principios · siempre"
    estándar = "senior-level o se itera"

    def approach(problema):
        núcleo = destilar(problema)
        return construir(núcleo) → iterar → repeat

    def __repr__():
        return "Construyo antes de que exista el manual."`}
              </pre>
            </Card>

            <div style={{
              display:"grid",gridTemplateColumns:"repeat(3,1fr)",gap:"1px",
              background:BORDER,border:`1px solid ${BORDER}`,borderRadius:"2px",
              overflow:"hidden",marginTop:"1rem",
            }}>
              {[
                {k:"PROYECTOS",v:"github.com/cortsito"},
                {k:"CONTACTO", v:"disponible para builds"},
                {k:"ESTADO",   v:"construyendo · activo"},
              ].map(({k,v}) => (
                <div key={k} style={{background:SURF,padding:"1rem",textAlign:"center"}}>
                  <div style={{fontFamily:FONT,fontSize:"0.55rem",color:DIM,letterSpacing:"0.18em",marginBottom:"6px"}}>{k}</div>
                  <div style={{fontFamily:FONT,fontSize:"0.65rem",color:EM2}}>{v}</div>
                </div>
              ))}
            </div>
          </div>
        )}

      </div>

      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@300;400;500;700&display=swap');
        * { box-sizing: border-box; }
        body { margin: 0; }
        @keyframes fadeIn { from{opacity:0;transform:translateY(6px)} to{opacity:1;transform:none} }
        @keyframes pulse {
          0%,100% { box-shadow: 0 0 0 0 rgba(16,185,129,0.4); }
          50% { box-shadow: 0 0 0 4px rgba(16,185,129,0); }
        }
        ::-webkit-scrollbar { width: 4px; }
        ::-webkit-scrollbar-track { background: ${BG}; }
        ::-webkit-scrollbar-thumb { background: rgba(16,185,129,0.3); border-radius: 2px; }
        button:hover { color: ${EM2} !important; }
      `}</style>
    </div>
  );
}
