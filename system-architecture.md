import { useState } from "react";

const NODES = {
  // ── INPUT LAYER ──
  camera:     { id:"camera",     label:"📷 Camera Capture",         sub:"High-res · Macro mode · Overlay guide",         layer:"input",      x:20,   y:80,  },
  voice:      { id:"voice",      label:"🎙 Voice Note",             sub:"Native mic · Max 120s · Auto-send",             layer:"input",      x:20,   y:190, },
  text:       { id:"text",       label:"⌨️ Text Query",             sub:"Drug name · Brand · Generic · Question",        layer:"input",      x:20,   y:300, },

  // ── CLIENT LAYER ──
  app:        { id:"app",        label:"📱 React Native App",       sub:"iOS + Android · TypeScript · Expo bare",        layer:"client",     x:220,  y:190, },
  biometric:  { id:"biometric",  label:"🔑 Biometric Auth",         sub:"FaceID · Fingerprint · Device-side only",       layer:"client",     x:220,  y:80,  },
  localstore: { id:"localstore", label:"💾 Local Cache",            sub:"Query history · Offline state · Keychain",      layer:"client",     x:220,  y:300, },

  // ── GATEWAY LAYER ──
  gateway:    { id:"gateway",    label:"🔐 API Gateway",            sub:"JWT · Rate limit · Routing · TLS 1.3",          layer:"gateway",    x:430,  y:190, },
  ratelimit:  { id:"ratelimit",  label:"⏱ Rate Limiter",            sub:"60 req/hr Professional · 5/15min auth",         layer:"gateway",    x:430,  y:80,  },
  auditlog:   { id:"auditlog",   label:"📋 Audit Logger",           sub:"Every request logged · NDPR required",          layer:"gateway",    x:430,  y:300, },

  // ── PROCESSING LAYER ──
  ocr:        { id:"ocr",        label:"👁 OCR Engine",             sub:"Google Vision · Handwriting · Confidence score",layer:"processing", x:640,  y:60,  },
  preprocess: { id:"preprocess", label:"🖼 Image Pre-processor",    sub:"Deskew · Contrast · Binarise · Normalise",      layer:"processing", x:640,  y:155, },
  stt:        { id:"stt",        label:"🗣 Whisper STT",            sub:"Self-hosted · whisper-medium · Nigerian EN",    layer:"processing", x:640,  y:250, },
  nlp:        { id:"nlp",        label:"🧠 NLP Drug Parser",        sub:"spaCy + scispaCy · NER · Entity extraction",    layer:"processing", x:640,  y:345, },

  // ── NORMALISATION ──
  rxnorm:     { id:"rxnorm",     label:"💊 RxNorm Normaliser",      sub:"Brand → Generic · Concept codes · NLM free",   layer:"normalise",  x:850,  y:250, },
  pii:        { id:"pii",        label:"🛡 PII Detector",           sub:"Strips patient identifiers · Pre-logging",      layer:"normalise",  x:850,  y:345, },

  // ── CLINICAL ENGINE ──
  engine:     { id:"engine",     label:"⚕️ Clinical Engine",        sub:"Parallel queries · Merge · Score · Synthesise", layer:"engine",     x:1060, y:190, },

  // ── DATABASE SOURCES ──
  openfda:    { id:"openfda",    label:"🏛 OpenFDA API",            sub:"Drug labels · Adverse events · Recalls · Free", layer:"database",   x:1270, y:60,  },
  rxnormdb:   { id:"rxnormdb",   label:"📗 RxNorm API",             sub:"Drug names · Mapping · NLM · No rate limit",   layer:"database",   x:1270, y:155, },
  ddi:        { id:"ddi",        label:"⚠️ DDI Database",           sub:"DrugBank/Lexicomp · Interactions · Paid",       layer:"database",   x:1270, y:250, },
  nafdac:     { id:"nafdac",     label:"🇳🇬 NAFDAC / WHO",          sub:"Nigerian registry · Essential medicines",        layer:"database",   x:1270, y:345, },
  drugbank:   { id:"drugbank",   label:"🔬 DrugBank",               sub:"Mechanisms · Targets · Pharmacokinetics",       layer:"database",   x:1270, y:440, },

  // ── AI SYNTHESIS ──
  llm:        { id:"llm",        label:"🤖 LLM Synthesis",          sub:"Claude Sonnet · Constrained prompt · Cite only",layer:"synthesis",  x:1060, y:400, },
  veracity:   { id:"veracity",   label:"✅ Veracity Scorer",        sub:"Citations / Claims ratio · Min 0.6 threshold",  layer:"synthesis",  x:850,  y:440, },

  // ── STORAGE LAYER ──
  postgres:   { id:"postgres",   label:"🗄 PostgreSQL",             sub:"RDS · User data · Audit trail · AES-256",       layer:"storage",    x:640,  y:480, },
  redis:      { id:"redis",      label:"⚡ Redis Cache",            sub:"ElastiCache · Drug hits · 24h TTL · <50ms",     layer:"storage",    x:430,  y:480, },
  s3:         { id:"s3",         label:"🔒 Encrypted S3",           sub:"af-south-1 · Prescriptions · 24h auto-delete",  layer:"storage",    x:220,  y:480, },

  // ── COMPLIANCE ──
  compliance: { id:"compliance", label:"🛡 Compliance Layer",       sub:"NDPR · PII strip · Data residency · Erasure",   layer:"compliance", x:430,  y:570, },
  ndpr:       { id:"ndpr",       label:"📜 NDPR Controls",          sub:"Consent · DPA · DPIA · Annual audit · NITDA",   layer:"compliance", x:220,  y:570, },

  // ── OUTPUT ──
  stream:     { id:"stream",     label:"📡 SSE Stream",             sub:"Server-Sent Events · Tokens appear in <1s",     layer:"output",     x:220,  y:370, },
  response:   { id:"response",   label:"✅ Clinical Response",      sub:"<5s · Cited · Scored · Disclaimed",             layer:"output",     x:20,   y:430, },
};

const EDGES = [
  {f:"camera",to:"app"},{f:"voice",to:"app"},{f:"text",to:"app"},
  {f:"biometric",to:"app"},{f:"app",to:"localstore"},
  {f:"app",to:"gateway"},
  {f:"ratelimit",to:"gateway"},{f:"gateway",to:"auditlog"},
  {f:"gateway",to:"preprocess"},{f:"gateway",to:"stt"},{f:"gateway",to:"nlp"},
  {f:"gateway",to:"s3"},
  {f:"preprocess",to:"ocr"},{f:"ocr",to:"nlp"},
  {f:"stt",to:"nlp"},
  {f:"nlp",to:"rxnorm"},{f:"nlp",to:"pii"},
  {f:"pii",to:"postgres"},
  {f:"rxnorm",to:"engine"},
  {f:"gateway",to:"redis"},{f:"redis",to:"engine"},
  {f:"engine",to:"openfda"},{f:"engine",to:"rxnormdb"},
  {f:"engine",to:"ddi"},{f:"engine",to:"nafdac"},{f:"engine",to:"drugbank"},
  {f:"engine",to:"llm"},
  {f:"llm",to:"veracity"},
  {f:"veracity",to:"postgres"},
  {f:"engine",to:"postgres"},
  {f:"gateway",to:"compliance"},{f:"compliance",to:"ndpr"},
  {f:"s3",to:"compliance"},
  {f:"llm",to:"stream"},{f:"stream",to:"response"},
  {f:"response",to:"app"},
];

const LAYER_META = {
  input:      { label:"INPUT",       color:"#38BDF8", desc:"3 input modalities" },
  client:     { label:"CLIENT",      color:"#818CF8", desc:"React Native app" },
  gateway:    { label:"GATEWAY",     color:"#A78BFA", desc:"Security & routing" },
  processing: { label:"PROCESSING",  color:"#F59E0B", desc:"OCR · STT · NLP" },
  normalise:  { label:"NORMALISE",   color:"#FB923C", desc:"RxNorm · PII strip" },
  engine:     { label:"ENGINE",      color:"#34D399", desc:"Clinical intelligence" },
  database:   { label:"DATABASES",   color:"#F87171", desc:"5 validated sources" },
  synthesis:  { label:"SYNTHESIS",   color:"#4ADE80", desc:"LLM · Veracity" },
  storage:    { label:"STORAGE",     color:"#22D3EE", desc:"Postgres · Redis · S3" },
  compliance: { label:"COMPLIANCE",  color:"#94A3B8", desc:"NDPR · Encryption" },
  output:     { label:"OUTPUT",      color:"#86EFAC", desc:"<5s clinical response" },
};

const DESCRIPTIONS = {
  camera:     "High-resolution native camera with a prescription-framing overlay guide. Macro mode enabled. No Telegram compression — full image quality preserved. Auto-capture triggers when framing quality threshold is met.",
  voice:      "Native microphone input. Waveform visualiser during recording. Maximum 120 seconds. Silence detection auto-sends after 3 seconds. Hands-free for sterile clinical environments.",
  text:       "Standard text input. Accepts drug names (brand or generic), clinical questions, and dosage queries. Available on the Free tier. No rate limit on text for Professional subscribers.",
  app:        "React Native application (TypeScript). Single codebase for iOS and Android. Expo bare workflow. Streaming response display — first tokens appear in under 1 second. Query history stored locally.",
  biometric:  "FaceID and Fingerprint authentication. Biometric data never leaves the device — handled entirely by the device OS. Token stored in Keychain (iOS) / Keystore (Android), never AsyncStorage.",
  localstore: "Local query history for quick reference. Offline state management. Secure token storage via react-native-keychain. No sensitive data persisted locally beyond session tokens.",
  gateway:    "Single entry point for all API requests. JWT token validation (15-min access tokens, 7-day refresh with rotation). TLS 1.3 in transit. Routes to appropriate processing pipeline by input type.",
  ratelimit:  "Redis-backed rate limiting. Auth endpoints: 5 requests per 15 minutes. Query endpoints: 60 requests per hour on Professional tier. Prevents abuse and controls API costs.",
  auditlog:   "Every authenticated request is logged — required for NDPR compliance. Logs: user ID, action type, IP address, timestamp, metadata. Retained for 365 days. No raw query text stored.",
  ocr:        "Google Cloud Vision API. Handles both printed and handwritten prescription text — critical for Nigerian clinical environments. Returns confidence score. If confidence < 80%, app prompts user to retake photo. Target: < 1.5 seconds.",
  preprocess: "Image normalisation pipeline before OCR. Steps: deskew (correct angled photos), contrast enhancement, binarisation, noise reduction. Runs server-side in parallel with image upload.",
  stt:        "Self-hosted OpenAI Whisper (whisper-medium model). Strong performance on Nigerian English accents and medical terminology. Runs on GPU server. 10-second voice note transcribed in under 2 seconds.",
  nlp:        "spaCy + scispaCy biomedical NER model. Extracts: drug names (generic and brand), dosage values, units, frequency, route of administration, patient parameters. Target: < 0.5 seconds.",
  rxnorm:     "Maps extracted drug names to standardised RxNorm concept codes. Ensures consistent querying across all database sources regardless of input spelling or brand name variation. NLM API — free.",
  pii:        "Regex and NER scan on all query text before logging. Detects and strips: NIN numbers, dates of birth, patient names, and other identifiers. Complies with NDPR data minimisation requirement.",
  engine:     "Core clinical intelligence orchestrator. Fires all database API calls simultaneously using asyncio.gather(). Merges results. Degrades gracefully if any source fails — never lets one failure kill the response. Passes merged data to LLM.",
  openfda:    "US FDA drug database. Free REST API. 120,000 requests/day with API key. Covers: drug labels, adverse event reports, recall notices, indications, contraindications. Primary free data source.",
  rxnormdb:   "National Library of Medicine standardised drug nomenclature. Free, no published rate limit — cache aggressively. Maps brand names to generics and vice versa. Used for cross-database consistency.",
  ddi:        "Drug-Drug Interaction engine. Paid license ($500–2,000/month — DrugBank or Lexicomp). Cross-references all identified drugs. Flags: contraindications, synergistic effects, antagonism, toxicity risk levels. The most important clinical differentiator.",
  nafdac:     "NAFDAC-registered Nigerian drug database and WHO Essential Medicines list. Critical for local market credibility. NAFDAC data requires partnership request. WHO data is freely available.",
  drugbank:   "Pharmacokinetics, drug targets, mechanism of action, and metabolic pathway data. Paid license. Enables deeper clinical explanations beyond interaction flags — adds clinical context.",
  llm:        "Claude Sonnet for synthesis. Operates under strict constraints: only synthesises from retrieved database data — never generates clinical claims from training knowledge. If no data found, responds 'Insufficient data — verify manually'. Streams response tokens.",
  veracity:   "Calculates the fraction of clinical claims in the response that have a traceable database citation. Formula: cited claims / total claims. Minimum display threshold: 0.60. Below threshold triggers a prominent user warning.",
  postgres:   "PostgreSQL 15 on AWS RDS. Stores: user accounts, subscription status, audit trail, professional verification records. AES-256 encryption at rest. Row-level encryption on PII fields. Never stores prescription images.",
  redis:      "Redis 7 on AWS ElastiCache. Caches drug query results for 24 hours. Cache hit rate target: 40–60% of queries. Brings repeat queries to under 50ms. Also handles rate limiting counters and session management.",
  s3:         "AWS S3 in af-south-1 (Cape Town) region — satisfies NDPR data residency. Prescription images stored with AES-256 encryption. S3 lifecycle policy auto-deletes after 24 hours. Never linked to patient identifiers in storage.",
  compliance: "Wraps the entire system. Responsibilities: TLS 1.3 in transit, AES-256 at rest, automatic PII detection, NDPR audit trail, data residency enforcement, session-based data retention, right-to-erasure workflow.",
  ndpr:       "Nigeria Data Protection Regulation 2019 controls. Requirements: explicit consent at registration, privacy policy, data minimisation, right to erasure within 30 days, annual NITDA audit via licensed DPCO, Data Protection Impact Assessment before v2.0 launch.",
  stream:     "Server-Sent Events streaming. First tokens delivered to the mobile app in under 1 second — user sees the response building in real time rather than waiting for the full response. Perceived speed is significantly faster than actual latency.",
  response:   "Structured clinical response. Contains: Drug Identification (brand + generic + RxNorm code), Interactions Found (with severity), Dosage Assessment vs therapeutic window, Clinical Flags, Veracity Score (0.0–1.0), Data Source citations, Mandatory disclaimer.",
};

const NW = 168, NH = 54;
function cx(n){ return n.x + NW/2; }
function cy(n){ return n.y + NH/2; }

export default function SOFHealthArch() {
  const [active, setActive] = useState(null);
  const [hoveredLayer, setHoveredLayer] = useState(null);
  const activeNode = active ? NODES[active] : null;

  const allX = Object.values(NODES).map(n => n.x);
  const allY = Object.values(NODES).map(n => n.y);
  const minX = Math.min(...allX) - 28;
  const minY = Math.min(...allY) - 28;
  const maxX = Math.max(...allX) + NW + 28;
  const maxY = Math.max(...allY) + NH + 60;

  return (
    <div style={{ minHeight:"100vh", background:"#080810", fontFamily:"'SF Mono','Fira Code','Courier New',monospace", color:"#E2E8F0", display:"flex", flexDirection:"column" }}>

      {/* Header */}
      <div style={{ padding:"18px 28px", borderBottom:"1px solid rgba(255,255,255,0.06)", background:"rgba(8,8,16,0.97)", position:"sticky", top:0, zIndex:100 }}>
        <div style={{ display:"flex", alignItems:"center", justifyContent:"space-between", marginBottom:12 }}>
          <div style={{ display:"flex", alignItems:"center", gap:12 }}>
            <div style={{ width:34, height:34, borderRadius:8, background:"linear-gradient(135deg,#34D399,#38BDF8)", display:"flex", alignItems:"center", justifyContent:"center", fontSize:16 }}>⚕️</div>
            <div>
              <div style={{ fontSize:14, fontWeight:700, color:"#F1F5F9" }}>SOFHealth — System Architecture</div>
              <div style={{ fontSize:9, color:"#475569", marginTop:2, letterSpacing:"0.05em" }}>
                {Object.keys(NODES).length} NODES · {EDGES.length} CONNECTIONS · CLICK ANY NODE TO INSPECT
              </div>
            </div>
          </div>
          {activeNode && (
            <div style={{ background:"rgba(52,211,153,0.1)", border:"1px solid rgba(52,211,153,0.2)", borderRadius:8, padding:"5px 12px", fontSize:10, color:"#34D399" }}>
              Inspecting: {activeNode.label}
            </div>
          )}
        </div>
        {/* Legend */}
        <div style={{ display:"flex", flexWrap:"wrap", gap:5 }}>
          {Object.entries(LAYER_META).map(([key, val]) => (
            <div key={key}
              onMouseEnter={() => setHoveredLayer(key)}
              onMouseLeave={() => setHoveredLayer(null)}
              style={{ display:"flex", alignItems:"center", gap:5, background: hoveredLayer===key ? "rgba(255,255,255,0.07)" : "rgba(255,255,255,0.02)", border:`1px solid ${val.color}25`, borderRadius:100, padding:"3px 9px", cursor:"default", transition:"all 0.15s" }}>
              <div style={{ width:5, height:5, borderRadius:1, background:val.color }} />
              <span style={{ fontSize:8.5, color:val.color, fontWeight:700, letterSpacing:"0.08em" }}>{val.label}</span>
              <span style={{ fontSize:8.5, color:"#334155" }}>· {val.desc}</span>
            </div>
          ))}
        </div>
      </div>

      <div style={{ display:"flex", flex:1, overflow:"hidden" }}>
        {/* Map */}
        <div style={{ flex:1, overflow:"auto", padding:"20px" }}>
          <svg viewBox={`${minX} ${minY} ${maxX-minX} ${maxY-minY}`} width="100%" style={{ display:"block", minWidth:1100 }}>
            <defs>
              {Object.entries(LAYER_META).map(([key,val]) => (
                <marker key={key} id={`arr-${key}`} markerWidth="6" markerHeight="6" refX="5" refY="3" orient="auto">
                  <path d="M0,0 L0,6 L6,3 z" fill={val.color+"80"} />
                </marker>
              ))}
              <filter id="glow"><feGaussianBlur stdDeviation="3" result="b"/><feMerge><feMergeNode in="b"/><feMergeNode in="SourceGraphic"/></feMerge></filter>
            </defs>

            {EDGES.map((e,i) => {
              const from = NODES[e.f], to = NODES[e.to];
              if (!from||!to) return null;
              const isActive = active && (e.f===active||e.to===active);
              const color = LAYER_META[from.layer]?.color||"#64748B";
              return <line key={i} x1={cx(from)} y1={cy(from)} x2={cx(to)} y2={cy(to)}
                stroke={isActive ? color : color+"22"} strokeWidth={isActive?1.5:0.7}
                strokeDasharray={isActive?"none":"3,4"}
                markerEnd={isActive?`url(#arr-${from.layer})`:undefined}
                filter={isActive?"url(#glow)":"none"}
                style={{transition:"all 0.2s"}} />;
            })}

            {Object.values(NODES).map(node => {
              const isSel = active===node.id;
              const isHov = hoveredLayer===node.layer;
              const lc = LAYER_META[node.layer]?.color||"#64748B";
              const opacity = hoveredLayer&&!isHov ? 0.15 : 1;
              return (
                <g key={node.id} transform={`translate(${node.x},${node.y})`}
                  style={{ cursor:"pointer", opacity, transition:"opacity 0.2s" }}
                  onClick={() => setActive(active===node.id ? null : node.id)}>
                  {isSel && <rect x={-5} y={-5} width={NW+10} height={NH+10} rx={14} fill={lc+"10"} filter="url(#glow)"/>}
                  <rect x={0} y={0} width={NW} height={NH} rx={9} fill={isSel?"#16213E":"#0E0E1A"} stroke={isSel?lc:lc+"30"} strokeWidth={isSel?1.5:0.7}/>
                  <rect x={0} y={0} width={3} height={NH} rx={2} fill={lc}/>
                  {isSel && <rect x={3} y={0} width={NW-3} height={1.5} rx={1} fill={lc} opacity={0.4}/>}
                  <text x={11} y={20} fontSize={9.5} fontWeight={700} fill={isSel?lc:"#E2E8F0"} fontFamily="SF Mono,monospace">{node.label}</text>
                  <text x={11} y={34} fontSize={7.8} fill="#475569" fontFamily="SF Mono,monospace">{node.sub}</text>
                  <circle cx={NW-10} cy={10} r={2.5} fill={lc} opacity={0.5}/>
                </g>
              );
            })}
          </svg>
        </div>

        {/* Detail Panel */}
        <div style={{ width:activeNode?290:0, minWidth:activeNode?290:0, transition:"all 0.25s ease", overflow:"hidden", borderLeft:activeNode?"1px solid rgba(255,255,255,0.06)":"none", background:"#0C0C18", display:"flex", flexDirection:"column" }}>
          {activeNode && (
            <div style={{ padding:"22px 18px", overflow:"auto", flex:1 }}>
              <div style={{ display:"inline-flex", alignItems:"center", gap:5, background:`${LAYER_META[activeNode.layer]?.color}10`, border:`1px solid ${LAYER_META[activeNode.layer]?.color}25`, borderRadius:100, padding:"3px 9px", marginBottom:14 }}>
                <div style={{ width:4, height:4, borderRadius:1, background:LAYER_META[activeNode.layer]?.color }}/>
                <span style={{ fontSize:8.5, color:LAYER_META[activeNode.layer]?.color, fontWeight:700, letterSpacing:"0.1em" }}>{LAYER_META[activeNode.layer]?.label} LAYER</span>
              </div>
              <div style={{ fontSize:14, fontWeight:700, color:"#F1F5F9", marginBottom:5, lineHeight:1.3 }}>{activeNode.label}</div>
              <div style={{ fontSize:9.5, color:"#475569", marginBottom:16, lineHeight:1.5 }}>{activeNode.sub}</div>
              <div style={{ fontSize:11.5, color:"#94A3B8", lineHeight:1.8, background:"rgba(255,255,255,0.02)", border:"1px solid rgba(255,255,255,0.05)", borderRadius:10, padding:"12px", marginBottom:18 }}>
                {DESCRIPTIONS[activeNode.id]||"No additional detail."}
              </div>
              <div style={{ fontSize:8.5, color:"#334155", letterSpacing:"0.1em", fontWeight:700, marginBottom:8 }}>CONNECTIONS</div>
              <div style={{ display:"flex", flexDirection:"column", gap:4 }}>
                {EDGES.filter(e=>e.f===activeNode.id||e.to===activeNode.id).map((e,i)=>{
                  const isOut = e.f===activeNode.id;
                  const otherId = isOut?e.to:e.f;
                  const other = NODES[otherId];
                  if (!other) return null;
                  const oc = LAYER_META[other.layer]?.color;
                  return (
                    <div key={i} onClick={()=>setActive(otherId)}
                      style={{ display:"flex", alignItems:"center", gap:7, padding:"6px 9px", background:"rgba(255,255,255,0.02)", border:`1px solid ${oc}18`, borderRadius:7, cursor:"pointer", transition:"background 0.15s" }}
                      onMouseEnter={ev=>ev.currentTarget.style.background="rgba(255,255,255,0.05)"}
                      onMouseLeave={ev=>ev.currentTarget.style.background="rgba(255,255,255,0.02)"}>
                      <span style={{ fontSize:9, color:oc, width:12, textAlign:"center" }}>{isOut?"→":"←"}</span>
                      <span style={{ fontSize:9.5, color:"#CBD5E1", flex:1 }}>{other.label}</span>
                      <div style={{ width:3, height:3, borderRadius:1, background:oc }}/>
                    </div>
                  );
                })}
              </div>
              <button onClick={()=>setActive(null)} style={{ marginTop:18, width:"100%", padding:"8px", background:"rgba(255,255,255,0.02)", border:"1px solid rgba(255,255,255,0.06)", borderRadius:7, color:"#475569", fontSize:10, cursor:"pointer", fontFamily:"inherit" }}>
                ✕ Close
              </button>
            </div>
          )}
        </div>
      </div>

      {/* Timing bar */}
      <div style={{ borderTop:"1px solid rgba(255,255,255,0.05)", padding:"9px 28px", background:"rgba(8,8,16,0.97)", display:"flex", alignItems:"center", flexWrap:"wrap", gap:5, fontSize:8.5, letterSpacing:"0.06em" }}>
        <span style={{ color:"#334155", marginRight:8 }}>PIPELINE TIMING:</span>
        {[
          {l:"Upload",t:"0.5s",c:"#38BDF8"},{l:"Pre-process",t:"0.3s",c:"#F59E0B"},
          {l:"OCR / STT",t:"1.0–1.5s",c:"#F59E0B"},{l:"NLP parse",t:"0.3–0.5s",c:"#FB923C"},
          {l:"DB queries ×5",t:"0.8–1.2s",c:"#F87171"},{l:"LLM synthesis",t:"1.0–1.5s",c:"#818CF8"},
          {l:"TOTAL",t:"< 5 sec",c:"#34D399"},
        ].map((s,i)=>(
          <div key={i} style={{ display:"flex", alignItems:"center" }}>
            {i>0 && <span style={{ color:"#1E293B", margin:"0 5px" }}>+</span>}
            <div style={{ background:`${s.c}10`, border:`1px solid ${s.c}28`, borderRadius:4, padding:"2px 8px", fontWeight:s.l==="TOTAL"?700:400 }}>
              <span style={{ color:"#475569" }}>{s.l}: </span>
              <span style={{ color:s.c }}>{s.t}</span>
            </div>
          </div>
        ))}
      </div>
    </div>
  );
}
