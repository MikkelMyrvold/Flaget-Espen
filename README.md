
import { useState, useEffect } from "react";
import { initializeApp } from "firebase/app";
import { getDatabase, ref, push, onValue, update, remove } from "firebase/database";

const firebaseConfig = {
  apiKey: "AIzaSyAvnxLkFpLlp6eZcK4A9G40Y8jdn5_fl_A",
  authDomain: "flaget-mannskap.firebaseapp.com",
  databaseURL: "https://flaget-mannskap-default-rtdb.europe-west1.firebasedatabase.app",
  projectId: "flaget-mannskap",
  storageBucket: "flaget-mannskap.firebasestorage.app",
  messagingSenderId: "596963772754",
  appId: "1:596963772754:web:b27337024ca2287f67928b"
};
const firebaseApp = initializeApp(firebaseConfig);
const db = getDatabase(firebaseApp);

// ── Ukeberegning ──────────────────────────────────────────────────────────────
function getWeekNumber(d) {
  const date = new Date(Date.UTC(d.getFullYear(), d.getMonth(), d.getDate()));
  const dayNum = date.getUTCDay() || 7;
  date.setUTCDate(date.getUTCDate() + 4 - dayNum);
  const yearStart = new Date(Date.UTC(date.getUTCFullYear(), 0, 1));
  return Math.ceil((((date - yearStart) / 86400000) + 1) / 7);
}

function getWeekDates(year, week) {
  const jan4 = new Date(year, 0, 4);
  const startOfWeek1 = new Date(jan4);
  startOfWeek1.setDate(jan4.getDate() - (jan4.getDay() || 7) + 1);
  const start = new Date(startOfWeek1);
  start.setDate(start.getDate() + (week - 1) * 7);
  const end = new Date(start);
  end.setDate(end.getDate() + 6);
  return {
    fra: start.toISOString().split("T")[0],
    til: end.toISOString().split("T")[0],
    label: `${start.toLocaleDateString("no-NO", { day: "numeric", month: "short" })} – ${end.toLocaleDateString("no-NO", { day: "numeric", month: "short", year: "numeric" })}`
  };
}

function getWeekOptions(year) {
  const weeks = [];
  for (let w = 1; w <= 52; w++) {
    const { label, fra, til } = getWeekDates(year, w);
    weeks.push({ week: w, label: `Uke ${w} · ${label}`, fra, til });
  }
  return weeks;
}

const now = new Date();
const currentYear = now.getFullYear();
const currentWeek = getWeekNumber(now);

// ── Data ──────────────────────────────────────────────────────────────────────
const PROSJEKTLEDERE = ["Espen", "Robert", "Jakob", "Einar", "Mikkel-test"];

const PROSJEKTER = [
  { id: "882", navn: "882 Kalkulasjon" }, { id: "999", navn: "999 Reklamasjoner" },
  { id: "1007", navn: "1007 Fravær" }, { id: "1008", navn: "1008 Drift" },
  { id: "1248", navn: "1248 Bingsveien 929" }, { id: "1403", navn: "1403 Hakavikveien Baksteval" },
  { id: "1452", navn: "1452 Stasjonsgata Forprosjekt" }, { id: "1687", navn: "1687 Åsensletta Mjøndalen" },
  { id: "1721", navn: "1721 Hemsedal Parkeringskjeller" }, { id: "1727", navn: "1727 Driftsbygning Eikerveien 95" },
  { id: "1742", navn: "1742 Hemsedal byggherre" }, { id: "1773", navn: "1773 Oppfølging lærlinger" },
  { id: "1784", navn: "1784 Leilighetsbygg A2 Hemsedal" }, { id: "1793", navn: "1793 OSO småjobber" },
  { id: "1834", navn: "1834 Solsetra Norefjell" }, { id: "1843", navn: "1843 Hovdeveien 4 Familiens Hus" },
  { id: "1850", navn: "1850 Hemsedal Leilighetsbygg A1" }, { id: "1861", navn: "1861 Stårputtveien 3, Oslo" },
  { id: "1864", navn: "1864 Hemsedal Leilighetsbygg A4" }, { id: "1865", navn: "1865 Hemsedal Leilighetsbygg A5" },
  { id: "1867", navn: "1867 SkiGeilo Lodge BT1" }, { id: "1870", navn: "1870 Norefjell Ski & Spa Bøseter" },
  { id: "1896", navn: "1896 Øvre Haugan gård. Prestfoss" }, { id: "1899", navn: "1899 Storåsveien 29B. Langhus" },
  { id: "1905", navn: "1905 Prosjekt SEAS" }, { id: "1909", navn: "1909 Kongensgate 2 Nova Park" },
  { id: "1911", navn: "1911 Sletthovdun 162. ÅL" }, { id: "1914", navn: "1914 Carl Stensehts vei 3" },
  { id: "1919", navn: "1919 Portåsveien 65. Mjøndalen" }, { id: "1920", navn: "1920 Hellik Teigen AS" },
  { id: "1921", navn: "1921 Skogstien 21 Blommenholm" }, { id: "1923", navn: "1923 Furnesbakken Stange Tekniske" },
  { id: "1927", navn: "1927 Fundamenter for adapteo" }, { id: "1929", navn: "1929 Vister P-kjeller AS" },
  { id: "1930", navn: "1930 Sameiet Vister A1-A3" }, { id: "1932", navn: "1932 Furnesbakken 3 overgangsbruer" },
  { id: "1933", navn: "1933 OK Lie Verksted Bingen" }, { id: "1935", navn: "1935 Vidsynveien 175 Norefjell" },
  { id: "1936", navn: "1936 Vesterøya massiv montering" }, { id: "1937", navn: "1937 Pentagon Massiv montering" },
  { id: "1938", navn: "1938 Skedsmo Skole hovedbygg" }, { id: "1939", navn: "1939 Skedsmo Skole massiv" },
  { id: "1945", navn: "1945 Driv Nebbenes" }, { id: "1946", navn: "1946 Bruksveien Snarøya" },
  { id: "1947", navn: "1947 Mjeltelivegen 63" }, { id: "1948", navn: "1948 Verksveien 116" },
  { id: "1951", navn: "1951 Langkroken 16c Nesbru" }, { id: "1952", navn: "1952 Argentitten 1 Kongsberg" },
  { id: "1958", navn: "1958 Støttemur Drammen Teigen" }, { id: "1960", navn: "1960 Åsendammen Mjøndalen" },
  { id: "1962", navn: "1962 RFD Mile" }, { id: "1963", navn: "1963 Renseanlegg HTAS" },
  { id: "1965", navn: "1965 Stasjonsgata 52-56 AS" }, { id: "1968", navn: "1968 Grettefoss Bru" },
  { id: "1971", navn: "1971 Jarmyrveien 3-9, Oslo" }, { id: "1973", navn: "1973 Staverhagan Billingstad" },
  { id: "1979", navn: "1979 Støagata 14 AS" }, { id: "1984", navn: "1984 Stertebakke Terrasse" },
  { id: "1987", navn: "1987 Øynebråten Anlegg Rammeavtale" }, { id: "1989", navn: "1989 Stømstangveien 46, Snarøya" },
  { id: "1993", navn: "1993 Vækerø Terrasse" }, { id: "1996", navn: "1996 Leirvangen 7, Hokksund" },
  { id: "1997", navn: "1997 DNT Bjørvasshytta" }, { id: "2001", navn: "2001 HTAS Administrasjonsbygg" },
  { id: "2003", navn: "2003 Grini Næringspark" }, { id: "2004", navn: "2004 Vestlia Hotel ny fløy" },
  { id: "2007", navn: "2007 Oterveien 14, Vikersund" }, { id: "2008", navn: "2008 Vestskogen - Betonmast" },
  { id: "2009", navn: "2009 Lampeland sentrumsutvikling" }, { id: "2011", navn: "2011 HTS produksjonshall Kongsberg" },
  { id: "2012", navn: "2012 Øvre Storgate Drammen Consto" }, { id: "2013", navn: "2013 Sideveisprosjekt Roa" },
  { id: "2015", navn: "2015 Søsteby Bru, Geithus" }, { id: "2016", navn: "2016 Kjelhovda Høydebasseng" },
  { id: "2017", navn: "2017 Sagatangen Massivtre" }, { id: "2018", navn: "2018 Tverrslag 1, Usta" },
  { id: "2020", navn: "2020 Verksted og vaskehall" }, { id: "2021", navn: "2021 Havsdalen Skifront" },
  { id: "192301", navn: "192301 K500 Helset Tekniske bygg" }, { id: "192302", navn: "192302 K501 Tangen Nord" },
  { id: "192303", navn: "192303 K502 Steinsrud Tekniske bygg" }, { id: "192304", navn: "192304 K503 Sørli Tekniske bygg" },
  { id: "1004", navn: "1004 Ett års befaring" }, { id: "2009-01", navn: "2009-01 LSU Prosjektutvikling" },
];

const KATEGORIER = [
  { id: "bas",            label: "Bas",            ikon: "★", beskrivelse: "Arbeidsformann / lagsbas" },
  { id: "fagarbeider",    label: "Fagarbeider",    ikon: "◆", beskrivelse: "Fagbrev / svennebrev" },
  { id: "laerling",       label: "Lærling",        ikon: "◇", beskrivelse: "Under opplæring" },
  { id: "hjelpearbeider", label: "Hjelpearbeider", ikon: "○", beskrivelse: "Ufaglært / generell" },
];

const KAT_LABEL = { bas: "Bas", fagarbeider: "Fagarbeider", laerling: "Lærling", hjelpearbeider: "Hjelpearbeider" };
const STATUS_LABEL = { sendt: "Sendt", bekreftet: "Bekreftet", avvist: "Avvist", arkivert: "Arkivert" };
const STATUS_FARGE = {
  sendt:     { bg: "#FFF8CC", color: "#886600" },
  bekreftet: { bg: "#E8F4EE", color: "#007833" },
  avvist:    { bg: "#FDEAEA", color: "#C0392B" },
  arkivert:  { bg: "#F0EDE8", color: "#888" },
};

const tomBestilling = () => ({
  prosjekt: "", prosjektleder: "Espen",
  fraUke: currentWeek, fraAar: currentYear,
  tilUke: currentWeek, tilAar: currentYear,
  linjer: KATEGORIER.map(k => ({ kategori: k.id, antall: 0, kompetanse: [] })),
  kommentar: "",
});

// ── CSS ───────────────────────────────────────────────────────────────────────
const css = `
  @import url('https://fonts.googleapis.com/css2?family=Ubuntu:wght@300;400;500;700&display=swap');
  *, *::before, *::after { box-sizing:border-box; margin:0; padding:0; }
  :root {
    --green:#007833; --green-dk:#005A25; --green-lt:#E8F4EE;
    --yellow:#FFDD00; --dark:#222426; --grey-lt:#dad9d7;
    --grey-bg:#F2F1EF; --white:#FFFFFF; --text:#222426; --muted:#666562;
    --radius:8px; --font:'Ubuntu', Arial, sans-serif;
  }
  body { background:var(--grey-bg); font-family:var(--font); color:var(--text); -webkit-font-smoothing:antialiased; }
  .app { max-width:430px; margin:0 auto; min-height:100vh; display:flex; flex-direction:column; }

  .header { background:var(--green); padding:44px 20px 16px; position:relative; overflow:hidden; }
  .header::before { content:''; position:absolute; top:-20px; right:-30px; width:160px; height:160px; background:rgba(255,255,255,0.07); transform:rotate(45deg); border-radius:20px; }
  .header::after { content:''; position:absolute; top:30px; right:50px; width:80px; height:80px; background:rgba(255,255,255,0.05); transform:rotate(45deg); border-radius:10px; }
  .header-logo { margin-bottom:10px; position:relative; z-index:1; }
  .header-sub { font-size:12px; color:rgba(255,255,255,0.7); position:relative; z-index:1; }

  .tabs { display:flex; background:var(--dark); }
  .tab { flex:1; padding:12px 6px; background:none; border:none; font-family:var(--font); font-size:11px; font-weight:500; letter-spacing:0.5px; text-transform:uppercase; color:rgba(255,255,255,0.45); cursor:pointer; border-bottom:3px solid transparent; transition:color 0.15s, border-color 0.15s; }
  .tab.active { color:var(--yellow); border-bottom-color:var(--yellow); }

  .content { padding:18px; flex:1; }
  .card { background:var(--white); border:1px solid var(--grey-lt); border-radius:var(--radius); margin-bottom:14px; overflow:hidden; box-shadow:0 1px 3px rgba(0,0,0,0.06); }
  .card-header { padding:11px 16px; background:var(--green); display:flex; align-items:center; gap:10px; }
  .card-ikon { color:var(--yellow); font-size:16px; }
  .card-tittel { font-size:14px; font-weight:700; color:var(--white); }
  .card-body { padding:16px; }

  .field { margin-bottom:14px; }
  .field:last-child { margin-bottom:0; }
  .label { display:block; font-size:11px; font-weight:500; letter-spacing:0.8px; text-transform:uppercase; color:var(--muted); margin-bottom:6px; }
  select, input[type=text], input[type=number], textarea {
    width:100%; padding:11px 14px; background:var(--grey-bg);
    border:1.5px solid var(--grey-lt); border-radius:var(--radius);
    font-family:var(--font); font-size:14px; color:var(--text);
    appearance:none; transition:border-color 0.15s; outline:none;
  }
  select:focus, input:focus, textarea:focus { border-color:var(--green); background:#fff; }
  textarea { resize:none; min-height:70px; }

  /* UKE VELGER */
  .uke-row { display:grid; grid-template-columns:1fr 1fr; gap:8px; }
  .uke-preview { margin-top:8px; padding:8px 12px; background:var(--green-lt); border:1px solid #b3d9c4; border-radius:var(--radius); font-size:12px; color:var(--green); font-weight:500; text-align:center; }

  /* PROSJEKT SØK */
  .search-wrap { position:relative; margin-bottom:6px; }
  .search-icon { position:absolute; left:12px; top:50%; transform:translateY(-50%); color:var(--muted); font-size:14px; pointer-events:none; }
  .search-input { padding-left:36px !important; }
  .prosjekt-dropdown { background:var(--white); border:1.5px solid var(--green); border-radius:var(--radius); max-height:200px; overflow-y:auto; margin-top:4px; box-shadow:0 4px 12px rgba(0,0,0,0.1); }
  .prosjekt-opt { padding:10px 14px; font-size:13px; cursor:pointer; border-bottom:1px solid var(--grey-lt); transition:background 0.1s; }
  .prosjekt-opt:last-child { border-bottom:none; }
  .prosjekt-opt:hover { background:var(--green-lt); color:var(--green); font-weight:500; }
  .prosjekt-valgt { padding:11px 14px; background:var(--green-lt); border:1.5px solid var(--green); border-radius:var(--radius); font-size:14px; font-weight:500; color:var(--green); display:flex; justify-content:space-between; align-items:center; cursor:pointer; }

  /* KATEGORI RADER */
  .kat-rad { border-bottom:1px solid var(--grey-lt); padding:14px 16px; }
  .kat-rad:last-child { border-bottom:none; }
  .kat-top { display:flex; align-items:center; gap:12px; }
  .kat-badge { width:36px; height:36px; border-radius:var(--radius); display:flex; align-items:center; justify-content:center; font-size:16px; flex-shrink:0; }
  .kat-badge.bas { background:#007833; color:#FFDD00; }
  .kat-badge.fagarbeider { background:#E8F4EE; color:#007833; }
  .kat-badge.laerling { background:#FFF8CC; color:#886600; }
  .kat-badge.hjelpearbeider { background:var(--grey-bg); color:var(--muted); border:1px solid var(--grey-lt); }
  .kat-label-wrap { flex:1; }
  .kat-navn { font-size:14px; font-weight:700; color:var(--text); }
  .kat-desc { font-size:11px; color:var(--muted); margin-top:1px; }
  .antall-wrap { display:flex; align-items:center; border:1.5px solid var(--grey-lt); border-radius:var(--radius); overflow:hidden; width:112px; }
  .step-btn { width:36px; height:36px; background:var(--grey-bg); border:none; font-size:20px; color:var(--muted); cursor:pointer; display:flex; align-items:center; justify-content:center; transition:all 0.1s; flex-shrink:0; }
  .step-btn.plus:hover { background:var(--green); color:white; }
  .antall-num { flex:1; text-align:center; font-size:18px; font-weight:700; color:var(--text); line-height:36px; }
  .antall-num.has-value { color:var(--green); }

  .komp-wrap { margin-top:10px; display:flex; flex-wrap:wrap; gap:6px; }
  .komp-chip { padding:5px 12px; border-radius:20px; border:1.5px solid var(--grey-lt); background:var(--grey-bg); font-size:12px; font-weight:500; color:var(--muted); cursor:pointer; transition:all 0.12s; user-select:none; }
  .komp-chip.valgt { background:var(--green); border-color:var(--green); color:white; }
  .komp-toggle-btn { font-size:11px; color:var(--green); font-weight:500; cursor:pointer; margin-top:8px; display:inline-block; text-decoration:underline; }
  .detalj-toggle { display:flex; align-items:center; gap:6px; cursor:pointer; font-size:12px; font-weight:500; color:var(--green); user-select:none; margin-top:10px; }
  .detalj-arrow { transition:transform 0.2s; font-size:10px; }
  .detalj-arrow.open { transform:rotate(90deg); }

  /* OPPSUMMERING */
  .opps-rad { display:flex; justify-content:space-between; align-items:flex-start; padding:13px 16px; border-bottom:1px solid var(--grey-lt); }
  .opps-rad:last-child { border-bottom:none; }
  .opps-kat { font-size:15px; font-weight:700; color:var(--text); }
  .opps-meta { font-size:12px; color:var(--muted); margin-top:3px; }
  .opps-antall { font-size:26px; font-weight:700; color:var(--green); line-height:1; }
  .opps-ant-label { font-size:10px; color:var(--muted); text-align:right; }

  /* HISTORIKK */
  .hist-card { background:var(--white); border:1px solid var(--grey-lt); border-left:4px solid var(--grey-lt); border-radius:var(--radius); margin-bottom:12px; overflow:hidden; box-shadow:0 1px 3px rgba(0,0,0,0.05); }
  .hist-card.sendt { border-left-color:var(--yellow); }
  .hist-card.bekreftet { border-left-color:var(--green); }
  .hist-header { padding:12px 16px; display:flex; justify-content:space-between; align-items:flex-start; border-bottom:1px solid var(--grey-lt); }
  .hist-prosjekt { font-size:14px; font-weight:700; color:var(--text); }
  .hist-dato { font-size:11px; color:var(--muted); margin-top:2px; }
  .hist-status { font-size:10px; font-weight:700; padding:4px 10px; border-radius:20px; white-space:nowrap; text-transform:uppercase; }
  .hist-status.sendt { background:#FFF8CC; color:#886600; }
  .hist-status.bekreftet { background:var(--green-lt); color:var(--green); }
  .hist-body { padding:12px 16px; }
  .hist-linje { display:flex; justify-content:space-between; font-size:13px; padding:4px 0; border-bottom:1px solid var(--grey-lt); }
  .hist-linje:last-child { border-bottom:none; }
  .hist-linje-label { color:var(--muted); }
  .tildelt-chip { display:inline-flex; align-items:center; gap:5px; background:var(--green-lt); border:1px solid #b3d9c4; border-radius:20px; padding:4px 10px; margin:3px; font-size:12px; color:var(--green); font-weight:500; }

  .btn-send { width:100%; padding:16px; background:var(--green); border:none; border-radius:var(--radius); font-family:var(--font); font-size:15px; font-weight:700; letter-spacing:0.5px; text-transform:uppercase; color:white; cursor:pointer; transition:background 0.15s; margin-top:6px; }
  .btn-send:hover { background:var(--green-dk); }
  .btn-send:disabled { background:var(--grey-lt); color:var(--muted); cursor:not-allowed; }
  .btn-reset { width:100%; padding:13px; background:none; border:1.5px solid var(--grey-lt); border-radius:var(--radius); font-family:var(--font); font-size:13px; font-weight:500; color:var(--muted); cursor:pointer; margin-top:10px; }

  .suksess-wrap { text-align:center; padding:50px 24px; animation:fadeIn 0.4s ease; }
  @keyframes fadeIn { from { opacity:0; transform:translateY(12px); } to { opacity:1; transform:translateY(0); } }
  .suksess-sirkel { width:80px; height:80px; background:var(--green); border-radius:50%; display:flex; align-items:center; justify-content:center; margin:0 auto 20px; font-size:32px; color:white; }
  .suksess-tittel { font-size:22px; font-weight:700; color:var(--text); margin-bottom:10px; }
  .suksess-sub { font-size:14px; color:var(--muted); margin-bottom:32px; line-height:1.6; }

  .footer-bar { background:var(--dark); padding:10px 20px; text-align:center; }
  .footer-bar-text { font-size:10px; color:rgba(255,255,255,0.3); letter-spacing:0.5px; }
  .empty-state { text-align:center; padding:50px 20px; color:var(--muted); }
`;

const LogoSVG = () => (
  <svg width="130" height="36" viewBox="0 0 130 36" fill="none">
    <rect width="36" height="36" rx="3" fill="#FFDD00"/>
    <polygon points="18,6 30,26 6,26" fill="#007833"/>
    <rect x="14" y="19" width="8" height="8" fill="#007833" transform="rotate(45 18 23)"/>
    <text x="44" y="26" fontFamily="Ubuntu, Arial, sans-serif" fontWeight="700" fontSize="20" fill="white" letterSpacing="0.5">FLAGET</text>
  </svg>
);

function ProsjektSøk({ value, onChange }) {
  const [søk, setSøk] = useState("");
  const [åpen, setÅpen] = useState(false);
  const valgt = PROSJEKTER.find(p => p.id === value);
  if (valgt) return (
    <div className="prosjekt-valgt" onClick={() => { onChange(""); setSøk(""); }}>
      <span>{valgt.navn}</span><span style={{ color: "var(--muted)", fontSize: 18 }}>✕</span>
    </div>
  );
  const filtrert = PROSJEKTER.filter(p =>
    p.navn.toLowerCase().includes(søk.toLowerCase()) || p.id.includes(søk)
  ).slice(0, 30);
  return (
    <div>
      <div className="search-wrap">
        <span className="search-icon">🔍</span>
        <input className="search-input" type="text" placeholder="Søk prosjektnr. eller navn…"
          value={søk} onChange={e => { setSøk(e.target.value); setÅpen(true); }} onFocus={() => setÅpen(true)} />
      </div>
      {åpen && søk.length > 0 && (
        <div className="prosjekt-dropdown">
          {filtrert.length === 0 && <div className="prosjekt-opt" style={{ color: "var(--muted)" }}>Ingen treff</div>}
          {filtrert.map(p => (
            <div key={p.id} className="prosjekt-opt" onClick={() => { onChange(p.id); setSøk(""); setÅpen(false); }}>{p.navn}</div>
          ))}
        </div>
      )}
    </div>
  );
}

const KOMPETANSER = ["HMS-kort","Stillas","Truck","Kran","Lift","Sveis","Gass","NEK400","Gravemaskin","Sprengning"];

function KategoriRad({ kat, linje, onChange }) {
  const [visMer, setVisMer] = useState(false);
  const [visKomp, setVisKomp] = useState(false);
  const settAntall = d => onChange({ ...linje, antall: Math.max(0, linje.antall + d) });
  const toggleKomp = k => onChange({ ...linje, kompetanse: linje.kompetanse.includes(k) ? linje.kompetanse.filter(x => x !== k) : [...linje.kompetanse, k] });
  return (
    <div className="kat-rad">
      <div className="kat-top">
        <div className={`kat-badge ${kat.id}`}>{kat.ikon}</div>
        <div className="kat-label-wrap">
          <div className="kat-navn">{kat.label}</div>
          <div className="kat-desc">{kat.beskrivelse}</div>
        </div>
        <div className="antall-wrap">
          <button className="step-btn" onClick={() => settAntall(-1)}>−</button>
          <div className={`antall-num ${linje.antall > 0 ? "has-value" : ""}`}>{linje.antall}</div>
          <button className="step-btn plus" onClick={() => settAntall(1)}>+</button>
        </div>
      </div>
      {linje.antall > 0 && (
        <>
          <div className="detalj-toggle" onClick={() => setVisMer(v => !v)}>
            <span className={`detalj-arrow ${visMer ? "open" : ""}`}>▶</span>
            Legg til kompetansekrav
          </div>
          {visMer && (
            <div style={{ marginTop: 10 }}>
              <div className="label">Kompetansekrav</div>
              <div className="komp-wrap">
                {(visKomp ? KOMPETANSER : KOMPETANSER.slice(0, 5)).map(k => (
                  <div key={k} className={`komp-chip ${linje.kompetanse.includes(k) ? "valgt" : ""}`} onClick={() => toggleKomp(k)}>{k}</div>
                ))}
              </div>
              <span className="komp-toggle-btn" onClick={() => setVisKomp(v => !v)}>{visKomp ? "Vis færre ▲" : "Vis alle ▼"}</span>
            </div>
          )}
        </>
      )}
    </div>
  );
}

// ── UkeVelger ─────────────────────────────────────────────────────────────────
function UkeVelger({ fraUke, fraAar, tilUke, tilAar, onChange }) {
  const aarOptions = [currentYear - 1, currentYear, currentYear + 1];
  const fraInfo = getWeekDates(fraAar, fraUke);
  const tilInfo = getWeekDates(tilAar, tilUke);
  return (
    <div>
      <div style={{ marginBottom: 12 }}>
        <div className="label">Fra uke</div>
        <div className="uke-row">
          <select value={fraUke} onChange={e => onChange({ fraUke: Number(e.target.value), fraAar, tilUke, tilAar })}>
            {Array.from({ length: 52 }, (_, i) => i + 1).map(w => (
              <option key={w} value={w}>Uke {w}</option>
            ))}
          </select>
          <select value={fraAar} onChange={e => onChange({ fraUke, fraAar: Number(e.target.value), tilUke, tilAar })}>
            {aarOptions.map(y => <option key={y} value={y}>{y}</option>)}
          </select>
        </div>
        <div className="uke-preview">📅 {fraInfo.label}</div>
      </div>
      <div>
        <div className="label">Til uke</div>
        <div className="uke-row">
          <select value={tilUke} onChange={e => onChange({ fraUke, fraAar, tilUke: Number(e.target.value), tilAar })}>
            {Array.from({ length: 52 }, (_, i) => i + 1).map(w => (
              <option key={w} value={w}>Uke {w}</option>
            ))}
          </select>
          <select value={tilAar} onChange={e => onChange({ fraUke, fraAar, tilUke, tilAar: Number(e.target.value) })}>
            {aarOptions.map(y => <option key={y} value={y}>{y}</option>)}
          </select>
        </div>
        <div className="uke-preview">📅 {tilInfo.label}</div>
      </div>
    </div>
  );
}

export default function App() {
  const [side, setSide] = useState("bestill");
  const [bestilling, setBestilling] = useState(tomBestilling());
  const [sendt, setSendt] = useState(false);
  const [historikk, setHistorikk] = useState([]);
  const [laster, setLaster] = useState(false);
  const [trekkTilbakeId, setTrekkTilbakeId] = useState(null); // bestilling som venter på bekreftelse

  useEffect(() => {
    const bestRef = ref(db, "bestillinger");
    return onValue(bestRef, snapshot => {
      const data = snapshot.val();
      if (data) {
        const liste = Object.entries(data)
          .map(([key, val]) => ({ ...val, firebaseKey: key }))
          .sort((a, b) => new Date(b.sendt) - new Date(a.sendt));
        setHistorikk(liste);
      } else setHistorikk([]);
    });
  }, []);

  // Filtrer historikk — prosjektleder ser kun egne bestillinger
  const minebestillinger = historikk.filter(h =>
    !bestilling.prosjektleder || h.prosjektleder === bestilling.prosjektleder
  );

  const oppdaterLinje = (idx, ny) => setBestilling(prev => {
    const linjer = [...prev.linjer]; linjer[idx] = ny; return { ...prev, linjer };
  });

  const harBestilling = bestilling.prosjekt && bestilling.prosjektleder && bestilling.linjer.some(l => l.antall > 0);
  const totalAntall = bestilling.linjer.reduce((s, l) => s + l.antall, 0);

  const sendBestilling = async () => {
    setLaster(true);
    const prosjektNavn = PROSJEKTER.find(p => p.id === bestilling.prosjekt)?.navn || bestilling.prosjekt;
    const fraInfo = getWeekDates(bestilling.fraAar, bestilling.fraUke);
    const tilInfo = getWeekDates(bestilling.tilAar, bestilling.tilUke);
    const ny = {
      prosjektId: bestilling.prosjekt,
      prosjektNavn,
      prosjektleder: bestilling.prosjektleder,
      sendt: new Date().toISOString(),
      status: "sendt",
      fraUke: bestilling.fraUke, fraAar: bestilling.fraAar,
      tilUke: bestilling.tilUke, tilAar: bestilling.tilAar,
      fraDato: fraInfo.fra, tilDato: tilInfo.til,
      periodeLabel: `Uke ${bestilling.fraUke}/${bestilling.fraAar} – Uke ${bestilling.tilUke}/${bestilling.tilAar}`,
      kommentar: bestilling.kommentar,
      linjer: bestilling.linjer.filter(l => l.antall > 0).map(l => ({
        kategori: l.kategori, antall: l.antall, kompetanse: l.kompetanse
      })),
      tildelt: [],
    };
    try {
      await push(ref(db, "bestillinger"), ny);
      setSendt(true);
    } catch (e) { alert("Feil ved sending: " + e.message); }
    setLaster(false);
  };

  const trekkTilbake = async (firebaseKey) => {
    try {
      await remove(ref(db, `bestillinger/${firebaseKey}`));
      setTrekkTilbakeId(null);
    } catch (e) { alert("Feil: " + e.message); }
  };

  return (
    <>
      <style>{css}</style>
      <div className="app">
        <div className="header">
          <div className="header-logo"><LogoSVG /></div>
          <div className="header-sub">Bestilling av mannskap — Espen</div>
        </div>
        <div className="tabs">
          <button className={`tab ${side === "bestill" ? "active" : ""}`} onClick={() => setSide("bestill")}>Ny bestilling</button>
          <button className={`tab ${side === "oppsummer" ? "active" : ""}`} onClick={() => { if (harBestilling) setSide("oppsummer"); }}>
            Oppsummering{totalAntall > 0 ? ` (${totalAntall})` : ""}
          </button>
          <button className={`tab ${side === "historikk" ? "active" : ""}`} onClick={() => setSide("historikk")}>
            Historikk{minebestillinger.length > 0 ? ` (${minebestillinger.length})` : ""}
          </button>
        </div>

        {side === "bestill" && !sendt && (
          <div className="content">
            <div className="card">
              <div className="card-header"><span className="card-ikon">▦</span><span className="card-tittel">Prosjekt og periode</span></div>
              <div className="card-body">
                <div className="field">
                  <label className="label">Søk og velg prosjekt</label>
                  <ProsjektSøk value={bestilling.prosjekt} onChange={v => setBestilling(b => ({ ...b, prosjekt: v }))} />
                </div>
                <div style={{ display:"flex", alignItems:"center", gap:8, padding:"10px 14px", background:"var(--green-lt)", border:"1.5px solid var(--green)", borderRadius:"var(--radius)", marginBottom:14 }}>
                  <span style={{ fontSize:16 }}>👷</span>
                  <span style={{ fontFamily:"var(--font)", fontSize:14, fontWeight:700, color:"var(--green)" }}>Prosjektleder: Espen</span>
                </div>
                <div className="field">
                  <UkeVelger
                    fraUke={bestilling.fraUke} fraAar={bestilling.fraAar}
                    tilUke={bestilling.tilUke} tilAar={bestilling.tilAar}
                    onChange={v => setBestilling(b => ({ ...b, ...v }))}
                  />
                </div>
              </div>
            </div>
            <div className="card">
              <div className="card-header"><span className="card-ikon">◉</span><span className="card-tittel">Mannskap</span></div>
              {KATEGORIER.map((kat, idx) => <KategoriRad key={kat.id} kat={kat} linje={bestilling.linjer[idx]} onChange={ny => oppdaterLinje(idx, ny)} />)}
            </div>
            <div className="card">
              <div className="card-header"><span className="card-ikon">✎</span><span className="card-tittel">Kommentar</span></div>
              <div className="card-body">
                <textarea placeholder="Spesielle krav, informasjon til administrasjonen…" value={bestilling.kommentar} onChange={e => setBestilling(b => ({ ...b, kommentar: e.target.value }))} />
              </div>
            </div>
            <button className="btn-send" disabled={!harBestilling} onClick={() => setSide("oppsummer")}>Se oppsummering →</button>
          </div>
        )}

        {side === "oppsummer" && !sendt && (
          <div className="content">
            <div className="card">
              <div className="card-header"><span className="card-ikon">▦</span><span className="card-tittel">{PROSJEKTER.find(p => p.id === bestilling.prosjekt)?.navn || "—"}</span></div>
              <div className="opps-rad"><div style={{ fontSize: 13, color: "var(--muted)" }}>
                Prosjektleder: <strong style={{ color: "var(--text)" }}>{bestilling.prosjektleder}</strong>
              </div></div>
              <div className="opps-rad"><div style={{ fontSize: 13, color: "var(--muted)" }}>
                Periode: <strong style={{ color: "var(--green)" }}>Uke {bestilling.fraUke}/{bestilling.fraAar} – Uke {bestilling.tilUke}/{bestilling.tilAar}</strong>
              </div></div>
              {bestilling.linjer.filter(l => l.antall > 0).map((l, i) => (
                <div key={i} className="opps-rad">
                  <div>
                    <div className="opps-kat">{KAT_LABEL[l.kategori]}</div>
                    {l.kompetanse.length > 0 && <div style={{ fontSize: 11, color: "var(--green)", marginTop: 3 }}>Krav: {l.kompetanse.join(", ")}</div>}
                  </div>
                  <div><div className="opps-antall">{l.antall}</div><div className="opps-ant-label">pers.</div></div>
                </div>
              ))}
              {bestilling.kommentar && <div className="opps-rad"><div style={{ fontSize: 13, color: "var(--muted)", fontStyle: "italic" }}>{bestilling.kommentar}</div></div>}
            </div>
            <button className="btn-send" onClick={sendBestilling} disabled={laster}>{laster ? "Sender…" : "Send bestilling"}</button>
            <button className="btn-reset" onClick={() => setSide("bestill")}>← Tilbake og endre</button>
          </div>
        )}

        {sendt && (
          <div className="content">
            <div className="suksess-wrap">
              <div className="suksess-sirkel">✓</div>
              <div className="suksess-tittel">Bestilling sendt!</div>
              <div className="suksess-sub">Administrasjonen har mottatt bestillingen og vil bekrefte så snart mannskap er tildelt.</div>
              <button className="btn-send" onClick={() => { setBestilling(tomBestilling()); setSendt(false); }}>Ny bestilling</button>
              <button className="btn-reset" onClick={() => { setSide("historikk"); setSendt(false); setBestilling(tomBestilling()); }}>Se historikk</button>
            </div>
          </div>
        )}

        {side === "historikk" && (
          <div className="content">
            {minebestillinger.length === 0 ? (
              <div className="empty-state">
                <div style={{ fontSize: 40, marginBottom: 14 }}>📋</div>
                <div style={{ fontSize: 16, fontWeight: 700, marginBottom: 6, color: "var(--text)" }}>Ingen bestillinger ennå</div>
                <div style={{ fontSize: 13 }}>Bestillinger du sender vises her</div>
              </div>
            ) : minebestillinger.map(h => {
              const sf = STATUS_FARGE[h.status] || STATUS_FARGE.sendt;
              const kanTrekkes = h.status === "sendt";
              return (
              <div key={h.firebaseKey} className="hist-card" style={{ borderLeft: `4px solid ${sf.color}` }}>
                <div className="hist-header">
                  <div>
                    <div className="hist-prosjekt">{h.prosjektNavn}</div>
                    <div className="hist-dato">{h.periodeLabel || ""}</div>
                    <div className="hist-dato">{new Date(h.sendt).toLocaleDateString("no-NO")}</div>
                  </div>
                  <div className="hist-status" style={{ background: sf.bg, color: sf.color }}>
                    {STATUS_LABEL[h.status] || h.status}
                  </div>
                </div>
                <div className="hist-body">
                  {h.linjer.map((l, i) => (
                    <div key={i} className="hist-linje">
                      <span style={{ fontWeight: 500 }}>{KAT_LABEL[l.kategori]}</span>
                      <span className="hist-linje-label">{l.antall} pers.</span>
                    </div>
                  ))}
                  {h.status === "avvist" && h.avvistKommentar && (
                    <div style={{ marginTop: 8, padding: "8px 0", borderTop: "1px solid var(--grey-lt)", fontSize: 12, color: "#C0392B", fontStyle: "italic" }}>
                      ✖ Avvist: {h.avvistKommentar}
                    </div>
                  )}
                  {h.tildelt?.length > 0 && (
                    <div style={{ marginTop: 8, paddingTop: 8, borderTop: "1px solid var(--grey-lt)" }}>
                      <div style={{ fontSize: 11, color: "var(--muted)", marginBottom: 6, fontWeight: 500, textTransform: "uppercase" }}>Tildelt</div>
                      {h.tildelt.map((t, i) => <div key={i} className="tildelt-chip">✓ {t.navn}</div>)}
                    </div>
                  )}
                  {kanTrekkes && (
                    <div style={{ marginTop: 10, paddingTop: 10, borderTop: "1px solid var(--grey-lt)" }}>
                      {trekkTilbakeId === h.firebaseKey ? (
                        <div>
                          <div style={{ fontSize: 12, color: "var(--muted)", marginBottom: 8 }}>Er du sikker på at du vil trekke tilbake bestillingen?</div>
                          <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 8 }}>
                            <button style={{ padding: 10, background: "#C0392B", border: "none", borderRadius: "var(--radius)", fontFamily: "var(--font)", fontSize: 13, fontWeight: 700, color: "white", cursor: "pointer" }}
                              onClick={() => trekkTilbake(h.firebaseKey)}>Ja, trekk tilbake</button>
                            <button style={{ padding: 10, background: "none", border: "1.5px solid var(--grey-lt)", borderRadius: "var(--radius)", fontFamily: "var(--font)", fontSize: 13, color: "var(--muted)", cursor: "pointer" }}
                              onClick={() => setTrekkTilbakeId(null)}>Avbryt</button>
                          </div>
                        </div>
                      ) : (
                        <button style={{ width: "100%", padding: 10, background: "none", border: "1.5px solid #E8A800", borderRadius: "var(--radius)", fontFamily: "var(--font)", fontSize: 13, fontWeight: 700, color: "#886600", cursor: "pointer" }}
                          onClick={() => setTrekkTilbakeId(h.firebaseKey)}>↩ Trekk tilbake bestilling</button>
                      )}
                    </div>
                  )}
                </div>
              </div>
              );
            })}
          </div>
        )}

        <div className="footer-bar"><div className="footer-bar-text">FLAGET AS · Varig Kvalitet · www.flagetas.no</div></div>
      </div>
    </>
  );
}
