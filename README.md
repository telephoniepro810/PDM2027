<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover" />
<title>Dashboard PDM — Besoin / Capacité / Delta</title>
<script src="https://cdn.tailwindcss.com"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/react/18.2.0/umd/react.production.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/react-dom/18.2.0/umd/react-dom.production.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/7.23.5/babel.min.js"></script>
<style>
  :root { box-sizing: border-box; padding-top: env(safe-area-inset-top, 0px); padding-bottom: env(safe-area-inset-bottom, 0px); }
  html { scroll-padding-top: env(safe-area-inset-top, 0px); }
  body { background:#fff; }
</style>
</head>
<body>

<div id="password-gate" style="position:fixed;inset:0;z-index:99999;background:#f8fafc;display:flex;align-items:center;justify-content:center;font-family:Arial,sans-serif;">
  <div style="width:min(92%,380px);background:white;border:1px solid #e5e7eb;border-radius:16px;padding:28px;box-shadow:0 10px 30px rgba(0,0,0,.08);text-align:center;">
    <h1 style="font-size:22px;margin:0 0 8px;">Accès protégé</h1>
    <p style="font-size:14px;color:#6b7280;margin:0 0 20px;">Entrez le mot de passe pour accéder au dashboard.</p>
    <input id="password-input" type="password" placeholder="Mot de passe"
      style="width:100%;box-sizing:border-box;padding:11px 12px;border:1px solid #d1d5db;border-radius:8px;margin-bottom:10px;outline:none;">
    <button id="password-button"
      style="width:100%;padding:11px 12px;border:0;border-radius:8px;background:#111827;color:white;font-weight:600;cursor:pointer;">
      Accéder
    </button>
    <p id="password-error" style="display:none;color:#b91c1c;font-size:13px;margin:12px 0 0;">Mot de passe incorrect.</p>
  </div>
</div>

<script>
(function () {
  const PASSWORD = "pilotekahia2027";
  const gate = document.getElementById("password-gate");
  const input = document.getElementById("password-input");
  const button = document.getElementById("password-button");
  const error = document.getElementById("password-error");

  function verifyPassword() {
    if (input.value === PASSWORD) {
      gate.style.display = "none";
      sessionStorage.setItem("dashboard_access", "granted");
    } else {
      error.style.display = "block";
      input.value = "";
      input.focus();
    }
  }

  if (sessionStorage.getItem("dashboard_access") === "granted") {
    gate.style.display = "none";
  }

  button.addEventListener("click", verifyPassword);
  input.addEventListener("keydown", function (event) {
    if (event.key === "Enter") verifyPassword();
  });

  input.focus();
})();
</script>

<div id="root"></div>
<script type="text/babel" data-presets="react">
const { useState, useMemo } = React;

const MOIS = ["Jan","Fev","Mar","Avr","Mai","Juin","Juil","Aout","Sept","Oct","Nov","Dec"];
const SITES = ['PTV', 'PTM', 'PTS'];
const STATUTS = ['CDI', 'CDD', 'ALT'];
const SITE_STATUTS = SITES.flatMap((s) => STATUTS.map((st) => `${s}_${st}`));

const PE_LIST = [
  { key: 'SNT', name: 'SNT GEN' }, { key: 'TRT', name: 'TRTLVINT' }, { key: 'VAC', name: 'VAC GEN' },
  { key: 'AEM', name: 'TLV AEM' }, { key: 'AMH', name: 'TLV AMH' }, { key: 'CCH', name: 'TLV CCH' },
  { key: 'NCD', name: 'TLV NCD' }, { key: 'MAV', name: 'MAV' }, { key: 'OPS', name: 'OPSURCO' },
  { key: 'GAM', name: 'GAMPART' }, { key: 'MUL', name: 'MULTI' }, { key: 'AST', name: 'ASSTECH' },
  { key: 'AMD', name: 'AMHAUD' },
  { key: 'TGG', name: 'TLV INT GG' },
  { key: 'EPI', name: 'TLV EPI' },
  { key: 'SP2', name: 'TLV SPE2' },
  { key: 'SPE', name: 'SPE2' },
];

const INIT_EFFECTIFS = {
  PTV: { CDI: [40,40,40,40,40,40,40,40,40,40,40,40], CDD: [15,15,15,5,5,6,3,1,1,14,17,16], ALT: [10,10,10,10,10,10,10,6,7,7,7,7] },
  PTM: { CDI: [9,9,9,11,11,11,11,11,11,11,11,11], CDD: [10,10,10,0,0,0,0,0,0,0,0,0], ALT: [1,1,1,1,1,1,1,1,4,4,4,4] },
  PTS: { CDI: [8,8,8,14,14,16,19,19,19,19,19,19], CDD: [10,10,10,0,0,0,0,0,0,9,9,9], ALT: [2,2,2,2,2,2,2,2,4,4,4,4] },
};

const INIT_HEURES = {
  CDI: [87.19,83.31,91.68,111.49,98.22,110.88,91.27,87.19,91.48,91.68,103.52,92.09],
  CDD: [100.96,96.46,106.16,129.09,113.72,128.38,105.69,100.96,105.92,106.16,119.87,106.63],
  ALT: [33.04,31.57,34.74,42.25,37.22,42.02,34.59,33.04,34.67,34.74,39.23,34.9],
};
const INIT_PRESENCE = { CDI: Array(12).fill(0.98), CDD: Array(12).fill(0.98), ALT: Array(12).fill(0.98) };

const INIT_PRODUCTIVITE = {
  SNT:[6.5,6.5,6,6,6,6,6,6,6,6,6,6], TRT:[5,5,4.5,4.5,4.5,4.5,4.5,4.5,4.5,4.5,4.5,4.5],
  VAC:[5,6,6,6,6,6,6,6,6,6,6,6], AEM:[7,7,7,7,7,7,7,7,7,7,7,7],
  AMH:[8,8,7,7,7,7,7,7,7,7,7,7], CCH:[6,6,6,6,6,6,6,6,6,6,6,6],
  NCD:[5,5,5,5,5,5,5,5,5,5,5,5], MAV:[5,5,5,5,5,5,5,5,5,5,5,5],
  OPS:[8,8,8,8,8,8,8,8,8,8,8,8], GAM:[13,13,13,10,10,10,10,10,10,10,10,10],
  MUL:[5,5,5,3,3,3,3,3,3,3,3,3], AST:[5,5,5,4,4,4,4,4,4,4,4,4],
  AMD:[9,9,9,7,7,7,7,7,7,7,7,7],
  TGG:[0,0,0,0,0,0,0,0,0,0,0,0],
  EPI:[0,0,0,0,0,0,0,0,0,0,0,0],
  SP2:[0,0,0,0,0,0,0,0,0,0,0,0],
  SPE:[0,0,0,0,0,0,0,0,0,0,0,0],
};

const INIT_APPELS = {
  SNT:[40537.8,27940.2,25399.8,21935,19009.8,18353,16094.8,13322.5,22527.5,24587.2,29656.5,31564.5],
  TRT:[11147.9,7683.6,6984.9,6032.1,5227.7,5047.1,4426.1,3663.7,6195.1,6761.5,8155.5,8680.2],
  VAC:[3400.2,4112,3325.2,2622.8,2116.5,1915,2275.2,1865,1437.5,936,622.5,984],
  AEM:[382.2,492,561.6,409.9,511.3,500.8,343.2,366.4,756.4,430.7,356.5,298.3],
  AMH:[6775.8,4682.5,4765.8,3623.8,3615,3727.8,3653.8,3363,3777.8,3588.5,4547.8,4264.2],
  CCH:[209.5,148.5,166.5,151.5,231,202,191,143,145,399,158.5,144],
  NCD:[27.5,26.4,29.4,35.2,30.4,452.7,519.7,328.3,859.7,797,1061.3,878],
  MAV:[0,0,0,0,0,0,0,0,400,400,350,320],
  OPS:[98.6,94.7,104.9,125.2,108.4,126.1,101.4,99.8,103.8,102.9,118,102.2],
  GAM:[48.9,47,51.9,61.7,53.6,62.1,50.3,49.5,51.4,50.9,58.2,50.6],
  MUL:[507.7,487,541.1,648.2,559.7,652.9,522.8,514.3,535,530.2,610.4,526.7],
  AST:[171.2,164.3,182.4,218.1,188.6,219.6,176.3,173.4,180.3,178.7,205.5,177.6],
  AMD:[18.3,17.7,19.3,22.6,19.9,22.7,18.8,18.5,19.1,19,21.4,18.9],
  TGG:[0,0,0,0,0,0,0,0,0,0,0,0],
  EPI:[0,0,0,0,0,0,0,0,0,0,0,0],
  SP2:[0,0,0,0,0,0,0,0,0,0,0,0],
  SPE:[0,0,0,0,0,0,0,0,0,0,0,0],
};

const INIT_ELIG = {
  PTV_CDI: { SNT:1,TRT:1,VAC:1,AEM:1,AMH:1,CCH:1,NCD:1,MAV:1,OPS:1,GAM:1,MUL:1,AST:1,AMD:1,TGG:0,EPI:0,SP2:0,SPE:0 },
  PTV_CDD: { SNT:1,TRT:1,VAC:1,AEM:1,AMH:1,CCH:1,NCD:1,MAV:1,OPS:1,GAM:1,MUL:1,AST:0,AMD:0,TGG:0,EPI:0,SP2:0,SPE:0 },
  PTV_ALT: { SNT:1,TRT:1,VAC:1,AEM:1,AMH:1,CCH:1,NCD:1,MAV:1,OPS:1,GAM:1,MUL:1,AST:0,AMD:0,TGG:0,EPI:0,SP2:0,SPE:0 },
  PTM_CDI: { SNT:0,TRT:0,VAC:0,AEM:0,AMH:0,CCH:0,NCD:0,MAV:0,OPS:0,GAM:0,MUL:0,AST:0,AMD:0,TGG:0,EPI:0,SP2:0,SPE:0 },
  PTM_CDD: { SNT:1,TRT:1,VAC:0,AEM:0,AMH:0,CCH:0,NCD:1,MAV:1,OPS:0,GAM:0,MUL:0,AST:0,AMD:0,TGG:0,EPI:0,SP2:0,SPE:0 },
  PTM_ALT: { SNT:0,TRT:0,VAC:0,AEM:0,AMH:0,CCH:0,NCD:0,MAV:0,OPS:0,GAM:0,MUL:0,AST:0,AMD:0,TGG:0,EPI:0,SP2:0,SPE:0 },
  PTS_CDI: { SNT:1,TRT:1,VAC:0,AEM:1,AMH:1,CCH:0,NCD:1,MAV:1,OPS:0,GAM:0,MUL:0,AST:0,AMD:0,TGG:0,EPI:0,SP2:0,SPE:0 },
  PTS_CDD: { SNT:1,TRT:1,VAC:0,AEM:0,AMH:0,CCH:0,NCD:1,MAV:1,OPS:0,GAM:0,MUL:0,AST:0,AMD:0,TGG:0,EPI:0,SP2:0,SPE:0 },
  PTS_ALT: { SNT:1,TRT:1,VAC:0,AEM:0,AMH:0,CCH:0,NCD:1,MAV:1,OPS:0,GAM:0,MUL:0,AST:0,AMD:0,TGG:0,EPI:0,SP2:0,SPE:0 },
};

const num = (v) => (Number.isFinite(v) ? Math.round(v * 10) / 10 : 0);
const fmt = (v) => num(v).toLocaleString('fr-FR');

function CellInput({ value, onChange, width = 44 }) {
  return (
    <input
      type="number"
      value={value}
      onChange={(e) => onChange(parseFloat(e.target.value) || 0)}
      style={{ width }}
      className="border border-gray-300 rounded px-1 py-0.5 text-xs text-right"
    />
  );
}

function setAt(arr, i, v) {
  const copy = [...arr];
  copy[i] = v;
  return copy;
}

function Dashboard() {
  const [effectifs, setEffectifs] = useState(INIT_EFFECTIFS);
  const [heures, setHeures] = useState(INIT_HEURES);
  const [presence, setPresence] = useState(INIT_PRESENCE);
  const [productivite, setProductivite] = useState(INIT_PRODUCTIVITE);
  const [appels, setAppels] = useState(INIT_APPELS);
  const [elig, setElig] = useState(INIT_ELIG);
  const [moisIdx, setMoisIdx] = useState(0);
  const [ongletActif, setOngletActif] = useState('resultats');

  const setEffCell = (site, statut, moisI, v) =>
    setEffectifs((p) => ({ ...p, [site]: { ...p[site], [statut]: setAt(p[site][statut], moisI, v) } }));
  const setHeuresCell = (statut, moisI, v) => setHeures((p) => ({ ...p, [statut]: setAt(p[statut], moisI, v) }));
  const setPresenceCell = (statut, moisI, v) => setPresence((p) => ({ ...p, [statut]: setAt(p[statut], moisI, v) }));
  const setProdCell = (pe, moisI, v) => setProductivite((p) => ({ ...p, [pe]: setAt(p[pe], moisI, v) }));
  const setAppelsCell = (pe, moisI, v) => setAppels((p) => ({ ...p, [pe]: setAt(p[pe], moisI, v) }));
  const toggleElig = (ss, pe) => setElig((p) => ({ ...p, [ss]: { ...p[ss], [pe]: p[ss][pe] ? 0 : 1 } }));

  const { synthese12, detailMois } = useMemo(() => {
    const synth = MOIS.map((m, i) => {
      const poolSite = {};
      SITES.forEach((s) => {
        let t = 0;
        STATUTS.forEach((st) => { t += (effectifs[s][st][i] || 0) * (heures[st][i] || 0) * (presence[st][i] || 0); });
        poolSite[s] = t;
      });
      const besoinHeures = {};
      PE_LIST.forEach(({ key }) => {
        const p = productivite[key][i] || 0;
        besoinHeures[key] = p > 0 ? (appels[key][i] || 0) / p : 0;
      });
      const capaciteTotale = SITES.reduce((a, s) => a + poolSite[s], 0);
      const besoinTotal = PE_LIST.reduce((a, { key }) => a + besoinHeures[key], 0);
      return { mois: m, capacite: capaciteTotale, besoin: besoinTotal, delta: capaciteTotale - besoinTotal };
    });

    const i = moisIdx;
    const poolSite = {};
    const poolSiteStatut = {};
    SITES.forEach((s) => {
      let t = 0;
      STATUTS.forEach((st) => {
        const c = (effectifs[s][st][i] || 0) * (heures[st][i] || 0) * (presence[st][i] || 0);
        poolSiteStatut[`${s}_${st}`] = c;
        t += c;
      });
      poolSite[s] = t;
    });
    const sitesEligiblesParPE = {};
    PE_LIST.forEach(({ key }) => {
      sitesEligiblesParPE[key] = SITES.filter((s) => STATUTS.some((st) => elig[`${s}_${st}`][key] === 1));
    });
    // Capacite disponible par PE = somme des heures des cases site_statut cochees pour ce PE.
    // Formule : CAPACITE_DISPO[pe] = SOMME( site_statut coche ) de EFFECTIF x HEURES/PERS x PRESENCE
    // (capacite "brute" si ce PE avait acces exclusif aux populations cochees ;
    // ne tient pas compte du partage avec d'autres PE sur la meme case - voir Synthese par site
    // pour le chiffre national non double-compte)
    const capaciteDisponibleParPE = {};
    PE_LIST.forEach(({ key }) => {
      let t = 0;
      SITE_STATUTS.forEach((ss) => { if (elig[ss][key] === 1) t += poolSiteStatut[ss]; });
      capaciteDisponibleParPE[key] = t;
    });
    // Formule : BESOIN_HEURES[pe] = APPELS_PREVUS[pe] / PRODUCTIVITE[pe]  (productivite = appels traites / heure)
    const besoinHeures = {};
    PE_LIST.forEach(({ key }) => {
      const p = productivite[key][i] || 0;
      besoinHeures[key] = p > 0 ? (appels[key][i] || 0) / p : 0;
    });
    const typeParPE = {};
    PE_LIST.forEach(({ key }) => {
      const n = sitesEligiblesParPE[key].length;
      typeParPE[key] = n === 0 ? 'AUCUN' : n === 1 ? 'MONO' : 'MULTI';
    });
    // Allocation monosite puis multisite (Delta = Capacite - Besoin, PDM section 21)
    const besoinMonoParSite = { PTV: 0, PTM: 0, PTS: 0 };
    PE_LIST.forEach(({ key }) => { if (typeParPE[key] === 'MONO') besoinMonoParSite[sitesEligiblesParPE[key][0]] += besoinHeures[key]; });
    const deltaMonoParSite = {};
    SITES.forEach((s) => { deltaMonoParSite[s] = poolSite[s] - besoinMonoParSite[s]; });
    const sitesMulti = new Set();
    let besoinMultiTotal = 0;
    PE_LIST.forEach(({ key }) => { if (typeParPE[key] === 'MULTI') { sitesEligiblesParPE[key].forEach((s) => sitesMulti.add(s)); besoinMultiTotal += besoinHeures[key]; } });
    const capaciteMultiDispo = [...sitesMulti].reduce((a, s) => a + deltaMonoParSite[s], 0);
    const deltaMulti = capaciteMultiDispo - besoinMultiTotal;
    const capaciteTotale = SITES.reduce((a, s) => a + poolSite[s], 0);
    const besoinTotal = PE_LIST.reduce((a, { key }) => a + besoinHeures[key], 0);

    return {
      synthese12: synth,
      detailMois: {
        poolSite, poolSiteStatut, capaciteDisponibleParPE, sitesEligiblesParPE, besoinHeures, typeParPE, besoinMonoParSite,
        deltaMonoParSite, sitesMulti, besoinMultiTotal, capaciteMultiDispo, deltaMulti,
        capaciteTotale, besoinTotal, deltaNational: capaciteTotale - besoinTotal,
      },
    };
  }, [effectifs, heures, presence, productivite, appels, elig, moisIdx]);

  const maxAbs = Math.max(...synthese12.map((s) => Math.max(s.capacite, s.besoin)), 1);

  return (
    <div className="max-w-6xl mx-auto p-4 text-sm space-y-6">
      <div>
        <h2 className="text-lg font-medium mb-1">Dashboard PDM — matrices mensuelles éditables (17 PE)</h2>
        <p className="text-gray-500 text-xs">Effectifs, heures/personne, présence, productivité et appels prévus sont éditables mois par mois. Tout se recalcule en direct et reste relié : Effectifs → Capacité, Appels prévus ÷ Productivité → Besoin, Compétences → Capacité disponible par PE.</p>
      </div>

      <div>
        <h3 className="font-medium mb-2">Vue nationale — Capacité vs Besoin vs Delta, 12 mois</h3>
        <div className="space-y-1">
          {synthese12.map((s) => (
            <div key={s.mois} className="flex items-center gap-2">
              <span className="w-10 text-xs text-gray-500">{s.mois}</span>
              <div className="flex-1 flex gap-0.5 h-4">
                <div className="bg-blue-400 rounded-sm" style={{ width: `${(s.capacite / maxAbs) * 100}%` }} title={`Capacité ${fmt(s.capacite)}h`} />
              </div>
              <div className="flex-1 flex gap-0.5 h-4">
                <div className="bg-yellow-400 rounded-sm" style={{ width: `${(s.besoin / maxAbs) * 100}%` }} title={`Besoin ${fmt(s.besoin)}h`} />
              </div>
              <span className={`w-20 text-right text-xs font-medium ${s.delta >= 0 ? 'text-green-700' : 'text-red-700'}`}>
                {s.delta >= 0 ? '+' : ''}{fmt(s.delta)}h
              </span>
            </div>
          ))}
        </div>
        <div className="flex gap-4 mt-2 text-xs text-gray-500">
          <span className="flex items-center gap-1"><span className="w-2.5 h-2.5 bg-blue-400 rounded-sm inline-block" />Capacité</span>
          <span className="flex items-center gap-1"><span className="w-2.5 h-2.5 bg-yellow-400 rounded-sm inline-block" />Besoin</span>
          <span className="flex items-center gap-1 text-green-700"><span className="w-2.5 h-2.5 bg-green-600 rounded-sm inline-block" />Delta positif</span>
          <span className="flex items-center gap-1 text-red-700"><span className="w-2.5 h-2.5 bg-red-500 rounded-sm inline-block" />Delta négatif</span>
        </div>
      </div>

      <div className="flex items-center gap-2">
        <span className="font-medium">Mois détaillé :</span>
        <select value={moisIdx} onChange={(e) => setMoisIdx(parseInt(e.target.value))} className="border border-gray-300 rounded px-2 py-1 text-sm">
          {MOIS.map((m, i) => <option key={m} value={i}>{m}</option>)}
        </select>
      </div>

      <div className="grid grid-cols-3 gap-3">
        <div className="bg-gray-50 rounded-lg p-3">
          <p className="text-xs text-gray-500">Capacité — {MOIS[moisIdx]}</p>
          <p className="text-xl font-medium text-blue-600">{fmt(detailMois.capaciteTotale)} h</p>
        </div>
        <div className="bg-gray-50 rounded-lg p-3">
          <p className="text-xs text-gray-500">Besoin (17 PE) — {MOIS[moisIdx]}</p>
          <p className="text-xl font-medium text-yellow-600">{fmt(detailMois.besoinTotal)} h</p>
        </div>
        <div className="bg-gray-50 rounded-lg p-3">
          <p className="text-xs text-gray-500">Delta — {MOIS[moisIdx]}</p>
          <p className={`text-xl font-medium ${detailMois.deltaNational >= 0 ? 'text-green-700' : 'text-red-700'}`}>
            {detailMois.deltaNational >= 0 ? '+' : ''}{fmt(detailMois.deltaNational)} h
          </p>
        </div>
      </div>

      <div className="flex gap-2 border-b border-gray-200">
        {[
          ['resultats', 'Résultats par PE'],
          ['effectifs', 'Effectifs (matrice 12 mois)'],
          ['productivite', 'Productivité / Appels (matrice 12 mois)'],
          ['competences', 'Compétences (matrice PE x site/statut)'],
        ].map(([id, label]) => (
          <button
            key={id}
            onClick={() => setOngletActif(id)}
            className={`px-3 py-2 text-xs ${ongletActif === id ? 'border-b-2 border-blue-600 font-medium' : 'text-gray-500'}`}
          >
            {label}
          </button>
        ))}
      </div>

      {ongletActif === 'resultats' && (
        <div>
          <p className="text-xs text-gray-400 mb-2">
            "Capacité disponible" = somme des heures des cases cochées dans l'onglet Compétences pour ce PE (formule : Σ EFFECTIF × HEURES/PERS × PRÉSENCE sur les cases cochées).
            "Besoin (h)" = Appels prévus ÷ Productivité (onglet Productivité / Appels). "Delta indicatif" = Capacité disponible − Besoin.
          </p>
          <table className="border-collapse w-full">
            <thead>
              <tr className="text-gray-500">
                <th className="text-left pb-1 pr-2 font-normal">PE</th>
                <th className="pb-1 pr-2 font-normal">Besoin (h)</th>
                <th className="pb-1 pr-2 font-normal">Capacité disponible (h)</th>
                <th className="pb-1 pr-2 font-normal">Delta indicatif (h)</th>
                <th className="pb-1 pr-2 font-normal">Type</th>
                <th className="pb-1 pr-2 font-normal">Site(s) éligible(s)</th>
              </tr>
            </thead>
            <tbody>
              {PE_LIST.map(({ key, name }) => {
                const deltaInd = detailMois.capaciteDisponibleParPE[key] - detailMois.besoinHeures[key];
                return (
                  <tr key={key} className="border-t border-gray-100">
                    <td className="py-1 pr-2 font-medium">{name}</td>
                    <td className="py-1 pr-2 text-right text-yellow-700">{fmt(detailMois.besoinHeures[key])}</td>
                    <td className="py-1 pr-2 text-right text-blue-700">{fmt(detailMois.capaciteDisponibleParPE[key])}</td>
                    <td className={`py-1 pr-2 text-right font-medium ${deltaInd >= 0 ? 'text-green-700' : 'text-red-700'}`}>
                      {deltaInd >= 0 ? '+' : ''}{fmt(deltaInd)}
                    </td>
                    <td className="py-1 pr-2">
                      <span className={`px-2 py-0.5 rounded text-xs ${detailMois.typeParPE[key] === 'MONO' ? 'bg-blue-50 text-blue-700' : 'bg-purple-50 text-purple-700'}`}>
                        {detailMois.typeParPE[key]}
                      </span>
                    </td>
                    <td className="py-1 pr-2 text-gray-500">{detailMois.sitesEligiblesParPE[key].join(', ') || '—'}</td>
                  </tr>
                );
              })}
            </tbody>
          </table>

          <h3 className="font-medium mt-4 mb-2">Synthèse par site — {MOIS[moisIdx]}</h3>
          <p className="text-xs text-gray-400 mb-2">Formule : Delta site = Capacité site − Besoin monosite du site. Formule multisite : Delta multisite = (Σ capacité restante des sites éligibles après monosite) − Besoin multisite total. Ce tableau n'additionne pas deux fois la même capacité, contrairement au détail par PE ci-dessus.</p>
          <table className="border-collapse w-full">
            <thead>
              <tr className="text-gray-500">
                <th className="text-left pb-1 pr-4 font-normal">Site</th>
                <th className="pb-1 pr-4 font-normal">Capacité (h)</th>
                <th className="pb-1 pr-4 font-normal">Besoin monosite (h)</th>
                <th className="pb-1 pr-4 font-normal">Delta après monosite (h)</th>
              </tr>
            </thead>
            <tbody>
              {SITES.map((s) => (
                <tr key={s} className="border-t border-gray-100">
                  <td className="py-1 pr-4 font-medium">{s}</td>
                  <td className="py-1 pr-4 text-right text-blue-700">{fmt(detailMois.poolSite[s])}</td>
                  <td className="py-1 pr-4 text-right text-yellow-700">{fmt(detailMois.besoinMonoParSite[s])}</td>
                  <td className={`py-1 pr-4 text-right font-medium ${detailMois.deltaMonoParSite[s] >= 0 ? 'text-green-700' : 'text-red-700'}`}>
                    {detailMois.deltaMonoParSite[s] >= 0 ? '+' : ''}{fmt(detailMois.deltaMonoParSite[s])}
                  </td>
                </tr>
              ))}
              <tr className="border-t border-gray-300 bg-gray-50">
                <td className="py-1 pr-4 font-medium">Multisite ({[...detailMois.sitesMulti].join(', ')})</td>
                <td className="py-1 pr-4 text-right text-blue-700">{fmt(detailMois.capaciteMultiDispo)}</td>
                <td className="py-1 pr-4 text-right text-yellow-700">{fmt(detailMois.besoinMultiTotal)}</td>
                <td className={`py-1 pr-4 text-right font-medium ${detailMois.deltaMulti >= 0 ? 'text-green-700' : 'text-red-700'}`}>
                  {detailMois.deltaMulti >= 0 ? '+' : ''}{fmt(detailMois.deltaMulti)}
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      )}

      {ongletActif === 'effectifs' && (
        <div className="space-y-4 overflow-x-auto">
          <p className="text-xs text-gray-400">Formule reliée : Capacité[site,mois] = Σ(statut) Effectif × Heures/pers × Présence — visible dans l'onglet "Résultats par PE" et dans la vue nationale ci-dessus.</p>
          {SITES.map((s) => (
            <div key={s}>
              <p className="font-medium mb-1">{s}</p>
              <table className="border-collapse text-xs">
                <thead>
                  <tr>
                    <th className="text-left pr-2 pb-1 text-gray-500 font-normal">Statut</th>
                    {MOIS.map((m) => <th key={m} className="px-1 pb-1 text-gray-500 font-normal">{m}</th>)}
                  </tr>
                </thead>
                <tbody>
                  {STATUTS.map((st) => (
                    <tr key={st}>
                      <td className="pr-2 py-0.5 font-medium">{st}</td>
                      {MOIS.map((m, i) => (
                        <td key={m} className="px-0.5 py-0.5">
                          <CellInput value={effectifs[s][st][i]} onChange={(v) => setEffCell(s, st, i, v)} />
                        </td>
                      ))}
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
          ))}
          <div>
            <p className="font-medium mb-1">Heures productives / personne</p>
            <table className="border-collapse text-xs">
              <thead>
                <tr>
                  <th className="text-left pr-2 pb-1 text-gray-500 font-normal">Statut</th>
                  {MOIS.map((m) => <th key={m} className="px-1 pb-1 text-gray-500 font-normal">{m}</th>)}
                </tr>
              </thead>
              <tbody>
                {STATUTS.map((st) => (
                  <tr key={st}>
                    <td className="pr-2 py-0.5 font-medium">{st}</td>
                    {MOIS.map((m, i) => (
                      <td key={m} className="px-0.5 py-0.5">
                        <CellInput value={heures[st][i]} onChange={(v) => setHeuresCell(st, i, v)} />
                      </td>
                    ))}
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
          <div>
            <p className="font-medium mb-1">Taux de présence</p>
            <table className="border-collapse text-xs">
              <thead>
                <tr>
                  <th className="text-left pr-2 pb-1 text-gray-500 font-normal">Statut</th>
                  {MOIS.map((m) => <th key={m} className="px-1 pb-1 text-gray-500 font-normal">{m}</th>)}
                </tr>
              </thead>
              <tbody>
                {STATUTS.map((st) => (
                  <tr key={st}>
                    <td className="pr-2 py-0.5 font-medium">{st}</td>
                    {MOIS.map((m, i) => (
                      <td key={m} className="px-0.5 py-0.5">
                        <CellInput value={presence[st][i]} onChange={(v) => setPresenceCell(st, i, v)} />
                      </td>
                    ))}
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
        </div>
      )}

      {ongletActif === 'productivite' && (
        <div className="space-y-4 overflow-x-auto">
          <p className="text-xs text-gray-400">Formule reliée : Besoin (h) = Appels prévus ÷ Productivité — visible dans l'onglet "Résultats par PE" et dans l'onglet "Compétences".</p>
          <div>
            <p className="font-medium mb-1">Productivité (appels/h)</p>
            <table className="border-collapse text-xs">
              <thead>
                <tr>
                  <th className="text-left pr-2 pb-1 text-gray-500 font-normal">PE</th>
                  {MOIS.map((m) => <th key={m} className="px-1 pb-1 text-gray-500 font-normal">{m}</th>)}
                </tr>
              </thead>
              <tbody>
                {PE_LIST.map(({ key, name }) => (
                  <tr key={key}>
                    <td className="pr-2 py-0.5 font-medium">{name}</td>
                    {MOIS.map((m, i) => (
                      <td key={m} className="px-0.5 py-0.5">
                        <CellInput value={productivite[key][i]} onChange={(v) => setProdCell(key, i, v)} />
                      </td>
                    ))}
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
          <div>
            <p className="font-medium mb-1">Appels prévus</p>
            <table className="border-collapse text-xs">
              <thead>
                <tr>
                  <th className="text-left pr-2 pb-1 text-gray-500 font-normal">PE</th>
                  {MOIS.map((m) => <th key={m} className="px-1 pb-1 text-gray-500 font-normal">{m}</th>)}
                </tr>
              </thead>
              <tbody>
                {PE_LIST.map(({ key, name }) => (
                  <tr key={key}>
                    <td className="pr-2 py-0.5 font-medium">{name}</td>
                    {MOIS.map((m, i) => (
                      <td key={m} className="px-0.5 py-0.5">
                        <CellInput value={appels[key][i]} onChange={(v) => setAppelsCell(key, i, v)} width={52} />
                      </td>
                    ))}
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
        </div>
      )}

      {ongletActif === 'competences' && (
        <div className="overflow-x-auto">
          <p className="text-xs text-gray-400 mb-2">
            Ne varie pas dans le temps (simplification assumée). Formule reliée : Capacité disponible[pe] = Σ (cases cochées) Effectif × Heures/pers × Présence.
            Les lignes "Capacité dispo (h)" et "Besoin (h)" ci-dessous se recalculent en direct pour le mois sélectionné ({MOIS[moisIdx]}) — décochez une case et regardez-les bouger.
          </p>
          <table className="border-collapse text-xs">
            <thead>
              <tr>
                <th className="text-left pr-2 pb-1 text-gray-500 font-normal sticky left-0 bg-white">Site_Statut</th>
                {PE_LIST.map(({ key, name }) => <th key={key} className="px-1 pb-1 text-gray-500 font-normal" title={name}>{key}</th>)}
              </tr>
            </thead>
            <tbody>
              {SITE_STATUTS.map((ss) => (
                <tr key={ss}>
                  <td className="pr-2 py-0.5 font-medium sticky left-0 bg-white">{ss}</td>
                  {PE_LIST.map(({ key }) => (
                    <td key={key} className="text-center px-1 py-0.5">
                      <input type="checkbox" checked={elig[ss][key] === 1} onChange={() => toggleElig(ss, key)} />
                    </td>
                  ))}
                </tr>
              ))}
              <tr className="border-t-2 border-gray-400 bg-blue-50">
                <td className="pr-2 py-1 font-medium sticky left-0 bg-blue-50">Capacité dispo (h) — {MOIS[moisIdx]}</td>
                {PE_LIST.map(({ key }) => (
                  <td key={key} className="text-center px-1 py-1 font-medium text-blue-700">{fmt(detailMois.capaciteDisponibleParPE[key])}</td>
                ))}
              </tr>
              <tr className="bg-yellow-50">
                <td className="pr-2 py-1 font-medium sticky left-0 bg-yellow-50">Besoin (h) — {MOIS[moisIdx]}</td>
                {PE_LIST.map(({ key }) => (
                  <td key={key} className="text-center px-1 py-1 text-yellow-700">{fmt(detailMois.besoinHeures[key])}</td>
                ))}
              </tr>
              <tr>
                <td className="pr-2 py-1 font-medium sticky left-0 bg-white">Delta indicatif (h) — {MOIS[moisIdx]}</td>
                {PE_LIST.map(({ key }) => {
                  const d = detailMois.capaciteDisponibleParPE[key] - detailMois.besoinHeures[key];
                  return <td key={key} className={`text-center px-1 py-1 font-medium ${d >= 0 ? 'text-green-700' : 'text-red-700'}`}>{d >= 0 ? '+' : ''}{fmt(d)}</td>;
                })}
              </tr>
            </tbody>
          </table>
        </div>
      )}

      <p className="text-xs text-gray-400">
        Rappel : 17 PE couverts. Besoin monosite = somme
        des PE monosite du site sans distinction fine du statut éligible.
      </p>

      <footer className="mt-10 pt-4 border-t border-gray-200 text-center text-xs text-gray-500">
        Yann Kahia - Chef de pilotage téléphonie commerciale - Tous droits réservés
      </footer>
    </div>
  );
}

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<Dashboard />);
</script>
</body>
</html>
