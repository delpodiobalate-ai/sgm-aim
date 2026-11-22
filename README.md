
n-bottom:8px;">Entrar ou Cadastrar Código de Acesso Pessoal</h3>
      <p style="font-size:12px; color:#475569; margin-bottom:10px;">Cada técnico usa o seu Nome e Código de Acesso para registar atividades com o seu nome automático.</p>
      <div class="form-row">
        <div><label>Nome Completo</label><input type="text" id="acesso-nome" value="Delpódio Eugénio Balate"></div>
        <div><label>Cargo</label>
          <select id="acesso-cargo">
            <option value="Estagiário" selected>Estagiário</option>
            <option value="Técnico">Técnico</option>
            <option value="Engenheiro">Engenheiro</option>
            <option value="Chefe do sector">Chefe do sector</option>
            <option value="Director técnico">Director técnico</option>
          </select>
        </div>
        <div><label>Código de Acesso (PIN)</label><input type="text" id="acesso-codigo" value="849122"></div>
      </div>
      <div style="display:flex; gap:8px; flex-wrap:wrap; align-items:center;">
        <button class="btn btn-ok" type="button" onclick="cadastrarOuEntrarComCodigo()">Ativar Este Acesso</button>
        <span id="acesso-status-msg" style="font-size:12px; color:#15803d; font-weight:600;"></span>
      </div>
    </div>

    <div class="card">
      <h3 style="margin-bottom:10px;">Equipa Cadastrada (10+ Técnicos)</h3>
      <div class="form-row">
        <div><label>Novo Técnico (Nome)</label><input type="text" id="tec-nome" placeholder="Nome completo"></div>
        <div><label>Cargo</label>
          <select id="tec-cargo">
            <option value="Técnico">Técnico</option>
            <option value="Engenheiro">Engenheiro</option>
            <option value="Chefe do sector">Chefe do sector</option>
            <option value="Director técnico">Director técnico</option>
            <option value="Estagiário">Estagiário</option>
          </select>
        </div>
        <div><label>Especialidade</label><input type="text" id="tec-esp" placeholder="Ex: Mecânica, AVAC"></div>
        <div><label>Código de Acesso</label><input type="text" id="tec-pin" placeholder="Ex: 112233"></div>
      </div>
      <button class="btn btn-ok" type="button" onclick="addMembro()" style="margin-bottom:12px;">+ Cadastrar Membro na Equipa</button>
      <div style="overflow-x:auto;">
        <table>
          <thead><tr><th>ID</th><th>Nome</th><th>Cargo</th><th>Especialidade</th><th>Código</th><th>Ações</th></tr></thead>
          <tbody id="lista-tecnicos"></tbody>
        </table>
      </div>
    </div>

    <div class="card" style="border-left:4px solid var(--ok);">
      <h3 style="color:var(--primary); margin-bottom:6px;">Ligação Firebase (Pré-Configurada para Tempo Real)</h3>
      <p style="font-size:12px; color:#475569; margin-bottom:10px;">A base <code>https://sgm-aim-default-rtdb.firebaseio.com</code> já vem ligada automaticamente para todos os dispositivos.</p>
      <div class="form-row">
        <div>
          <label>Provedor</label>
          <select id="cloud-provider" onchange="salvarConfigNuvem()">
            <option value="firebase" selected>Firebase Realtime Database</option>
            <option value="none">Apenas Local</option>
          </select>
        </div>
        <div>
          <label>URL Base Firebase</label>
          <input type="text" id="cloud-url" value="https://sgm-aim-default-rtdb.firebaseio.com" onchange="salvarConfigNuvem()">
        </div>
        <div>
          <label>Caminho na Base</label>
          <input type="text" id="cloud-path" value="sgm_aim_estado" onchange="salvarConfigNuvem()">
        </div>
      </div>
      <div style="display:flex; gap:8px; flex-wrap:wrap; align-items:center;">
        <button class="btn btn-ok" type="button" onclick="cloudPush()">Enviar Dados para Firebase</button>
        <button class="btn" type="button" onclick="cloudPull()">Atualizar Dados do Firebase</button>
        <small id="cloud-last-sync" style="color:#334155;"></small>
      </div>
    </div>
  </div>

</div>

<!-- MODAL FOTO -->
<div id="foto-modal" onclick="fecharModalFoto()" style="display:none; position:fixed; inset:0; background:rgba(15,23,42,0.82); z-index:999; align-items:center; justify-content:center; padding:20px;">
  <div style="background:#fff; padding:14px; border-radius:8px; max-width:90vw; max-height:90vh; text-align:center;" onclick="event.stopPropagation()">
    <img id="foto-modal-img" src="" alt="Foto ampliada" style="max-width:100%; max-height:76vh; border-radius:6px; display:block; margin:0 auto 10px;">
    <button class="btn btn-sm btn-dark" type="button" onclick="fecharModalFoto()">Fechar</button>
  </div>
</div>

<script>
const STORAGE_KEY = 'sgm_aim_v50_state';
const DIAS = 31;
const DEFAULT_FIREBASE_URL = 'https://sgm-aim-default-rtdb.firebaseio.com';
const DEFAULT_FIREBASE_PATH = 'sgm_aim_estado';

const CATEGORIAS = [
  'VIATURAS','SLCI','EQUIPAMENTOS DE LIMPEZA DAS ÁREAS OPERACIONAIS',
  'ESCADAS ROLANTES','ELEVADOR','PONTES DE EMBARQUE','TAPETE ROLANTE',
  'CHILLERS','UTA','AC'
];

const EQUIPAMENTOS_BASE = [
  [1, 'MAHINDRA AMS-769-MC', 'VIATURAS', '=', ''],
  [2, 'MAHINDRA AMS-748- MC', 'VIATURAS', '=', ''],
  [3, 'MAHINDRA AMS-770- MC', 'VIATURAS', '=', ''],
  [4, 'MAHINDRA AMS-760- MC', 'VIATURAS', '=', ''],
  [5, 'MAHINDRA AMS-774- MC', 'VIATURAS', '=', ''],
  [6, 'MAHINDRA AMS-783- MC', 'VIATURAS', '=', ''],
  [7, 'MAHINDRA AMQ-423- MC', 'VIATURAS', '=', ''],
  [8, 'MAHINDRA ANW-091-MC', 'VIATURAS', '=', ''],
  [9, 'NISSAN MMS-50-96', 'VIATURAS', 'X', 'A1)'],
  [10, 'NISSAN ABY-628-MP', 'VIATURAS', '=', ''],
  [11, 'NISSAN ACQ-955-MP', 'VIATURAS', 'X', 'A2)'],
  [12, 'NISSAN ACQ-962-MP', 'VIATURAS', 'X', 'A3)'],
  [13, 'Nissan ADG-515-MP', 'VIATURAS', '=', ''],
  [14, 'Nissan ADC-725-MP', 'VIATURAS', 'X', 'A4)'],
  [15, 'NISSAN ACO-187-MP', 'VIATURAS', 'X', 'A5)'],
  [16, 'NISSAN AGX-553-MP', 'VIATURAS', 'X', 'A6)'],
  [17, 'NISSAN ADQ-806-MP', 'VIATURAS', 'X', 'A7)'],
  [18, 'TOYOTA ANR-668-MC', 'VIATURAS', '=', ''],
  [19, 'TOYOTA AOF-241-MC', 'VIATURAS', '=', ''],
  [20, 'TOYOTA AOE-055-MC', 'VIATURAS', '=', ''],
  [21, 'TOYOTA FALLOW ME AAO-909-MP', 'VIATURAS', '=', ''],
  [22, 'TOYOTA FALLOW ME AFQ-491-MP', 'VIATURAS', '=', ''],
  [23, 'NISSAN AFK-460-MP', 'VIATURAS', 'X', 'A7)'],
  [24, 'M. BENZ AAJ-965-MC MARCÉ-SLCI', 'SLCI', '=', ''],
  [25, 'M. BENZ AAE-587-CM ROSENBAUER-SLCI', 'SLCI', '=', ''],
  [26, 'M. BENZ AAJ-966-MC MARCÉ-SLCI', 'SLCI', 'X', 'A8﴿'],
  [27, 'NISSAN HARDBODY AMBULANCIA AEU-072-MP', 'SLCI', '=', ''],
  [28, 'SCANIA AMX -389-MC', 'SLCI', '=', ''],
  [29, 'TRACTOR MF MLX-63-56', 'EQUIPAMENTOS DE LIMPEZA DAS ÁREAS OPERACIONAIS', 'X', 'B1﴿'],
  [30, 'TRACTOR MF MBP-21-46', 'EQUIPAMENTOS DE LIMPEZA DAS ÁREAS OPERACIONAIS', 'X', ''],
  [31, 'TRACTOR NEW HOLLAND ABT-525-MC', 'EQUIPAMENTOS DE LIMPEZA DAS ÁREAS OPERACIONAIS', '=', ''],
  [32, 'TRACTOR VALTRA MLZ-77-03', 'EQUIPAMENTOS DE LIMPEZA DAS ÁREAS OPERACIONAIS', '=', 'B2)'],
  [33, 'CAPINADEIRA GALUCHO 1', 'EQUIPAMENTOS DE LIMPEZA DAS ÁREAS OPERACIONAIS', 'X', 'B3)'],
  [34, 'CAPINADEIRA GALUCHO 2', 'EQUIPAMENTOS DE LIMPEZA DAS ÁREAS OPERACIONAIS', 'X', 'B4﴿'],
  [35, 'CAPINADEIRA JOPER', 'EQUIPAMENTOS DE LIMPEZA DAS ÁREAS OPERACIONAIS', '=', ''],
  [36, 'ESCADA ROLANTE CHECK IN IPT', 'ESCADAS ROLANTES', '=', ''],
  [37, 'ESCADA ROLANTE DESEMBARQUE IPT', 'ESCADAS ROLANTES', '=', ''],
  [38, 'ESCADA ROLANTE REMOTA IPT', 'ESCADAS ROLANTES', '=', ''],
  [39, 'ESCADA ROLANTE EMBARQUE DPT', 'ESCADAS ROLANTES', '=', ''],
  [40, 'ESCADA ROLANTE EMBARQUE REMOTO DPT', 'ESCADAS ROLANTES', '=', ''],
  [41, 'ESCADA ROLANTE PÚBLICO 1', 'ESCADAS ROLANTES', '=', ''],
  [42, 'ESCADA ROLANTE PÚBLICO 2', 'ESCADAS ROLANTES', '=', ''],
  [43, 'ESCADA ROLANTE DESEMBARQUE DPT', 'ESCADAS ROLANTES', '=', ''],
  [44, 'ELEVADOR S. HONRA', 'ELEVADOR', '=', ''],
  [45, 'ELEVADOR EMB. DPT', 'ELEVADOR', '=', ''],
  [46, 'ELEVADOR DESEMB. DPT', 'ELEVADOR', '=', ''],
  [47, 'ELEVADOR PUBLICO DPT', 'ELEVADOR', '=', ''],
  [48, 'ELEVADOR CHECK-IN IPT', 'ELEVADOR', '=', ''],
  [49, 'ELEVADOR DESEMBARQUE IPT', 'ELEVADOR', '=', ''],
  [50, 'ELEVADOR REMOTA IPT', 'ELEVADOR', 'X', 'C1)'],
  [51, 'ELEVADOR TWR', 'ELEVADOR', '=', ''],
  [52, 'PONTE DE EMB. 2A IPT', 'PONTES DE EMBARQUE', '=', ''],
  [53, 'PONTE DE EMB. 2B IPT', 'PONTES DE EMBARQUE', 'X', 'D1)'],
  [54, 'PONTE DE EMB. 3A IPT', 'PONTES DE EMBARQUE', '=', ''],
  [55, 'PONTE DE EMB. 3B IPT', 'PONTES DE EMBARQUE', '=', 'D2)'],
  [56, 'PONTE DE EMB. 08 DPT', 'PONTES DE EMBARQUE', 'X', 'D3)'],
  [57, 'PONTE DE EMB. 09 DPT', 'PONTES DE EMBARQUE', '=', ''],
  [58, 'TAPETE ROLANTE DESEMBARQUE 1 IPT', 'TAPETE ROLANTE', '=', ''],
  [59, 'TAPETE ROLANTE DESEMBARQUE 2 IPT', 'TAPETE ROLANTE', '=', ''],
  [60, 'TAPETE ROLANTE CHECK IN 1 IPT', 'TAPETE ROLANTE', '=', ''],
  [61, 'TAPETE ROLANTE CHECK IN 2 IPT', 'TAPETE ROLANTE', '=', ''],
  [62, 'TAPETE ROLANTE DESEMBARQUE 1 DPT', 'TAPETE ROLANTE', '=', ''],
  [63, 'TAPETE ROLANTE DESEMBARQUE 2 DPT', 'TAPETE ROLANTE', 'X', 'E1)'],
  [64, 'TAPETE ROLANTE CHECK IN 1 DPT', 'TAPETE ROLANTE', '=', ''],
  [65, 'TAPETE ROLANTE CHECK IN 2 DPT', 'TAPETE ROLANTE', '=', ''],
  [66, 'CHILLERS IPT SLEVE I', 'CHILLERS', '=', ''],
  [67, 'CHILLERS IPT SLEVE II', 'CHILLERS', '=', ''],
  [68, 'CHILLERS IPT SLEVE III', 'CHILLERS', '=', ''],
  [69, 'CHILLERS IPT SLEVE IV', 'CHILLERS', '=', ''],
  [70, 'CHILLERS IPT SLEVE V', 'CHILLERS', 'X', ''],
  [71, 'CHILLERS IPT MASTER VI', 'CHILLERS', 'X', ''],
  [72, 'CHILLERS DPT SLEVE I', 'CHILLERS', '=', ''],
  [73, 'CHILLERS DPT SLEVE II', 'CHILLERS', 'X', 'F1)'],
  [74, 'CHILLERS DPT SLEVE III', 'CHILLERS', '=', ''],
  [75, 'CHILLERS DPT SLEVE IV', 'CHILLERS', '=', ''],
  [76, 'CHILLERS DPT SLEVE V', 'CHILLERS', '=', ''],
  [77, 'CHILLERS DPT MASTER VI', 'CHILLERS', '=', 'F2)'],
  [78, 'UTA IPT 1 EMBARQUE', 'UTA', '=', ''],
  [79, 'UTA IPT 2 EMBARQUE', 'UTA', 'X', 'F﴿'],
  [80, 'UTA IPT 3 CHECK IN', 'UTA', '=', ''],
  [81, 'UTA IPT 4 CORREDOR', 'UTA', 'X', 'F1)'],
  [82, 'UTA IPT 5 CORREDOR', 'UTA', '=', ''],
  [83, 'UTA IPT 6 DESEMBARQUE', 'UTA', '=', ''],
  [84, 'UTA IPT 7 EMBARQUE E SALA FLAMINGO', 'UTA', '=', ''],
  [85, 'UTA DPT 1 CORREDOR', 'UTA', '=', ''],
  [86, 'UTA DPT 2 CHECK IN', 'UTA', '=', ''],
  [87, 'UTA DPT 3 EMBARQUE', 'UTA', '=', ''],
  [88, 'UTA DPT 4 EMBARQUE', 'UTA', '=', 'F2)'],
  [89, 'UTA DPT 5 EMBARQUE REMOTO', 'UTA', '=', ''],
  [90, 'UTA DPT 6 CORREDOR', 'UTA', '=', ''],
  [91, 'UTA DPT 7 DESEMBARQU', 'UTA', '=', ''],
  [92, 'UTA DPT 8 ZONA PUBLICA 1° ANDAR', 'UTA', '=', ''],
  [93, "AC´s IPT", 'AC', '=', ''],
  [94, "AC´s DPT", 'AC', '=', ''],
  [95, "AC´s TORRE", 'AC', '=', ''],
  [96, "AC´s SALA VIP", 'AC', '=', ''],
  [97, "AC´s T. CARGA", 'AC', '=', ''],
  [98, "AC´s SLCI E ZONA OFICINAL", 'AC', '=', ''],
  [99, "AC´s SALAO DE HONRA", 'AC', '=', ''],
  [100, "AC´s VOR/DME", 'AC', 'X', 'G)'],
  [101, "AC´s GLIDE PATH", 'AC', '=', ''],
  [102, "AC´s LOCALIZER", 'AC', '=', ''],
  [103, "AC´s SALA TÉCNICA", 'AC', '=', '']
];

function gerarIdPortatil(nome, codigo) {
  const base = String(nome || '').trim().toUpperCase() + '|' + String(codigo || '').trim();
  let hash = 2166136261;
  for (let i = 0; i < base.length; i++) {
    hash ^= base.charCodeAt(i);
    hash = Math.imul(hash, 16777619);
  }
  return 'AIM-' + (hash >>> 0).toString(36).toUpperCase();
}

function addDaysISO(isoDate, days) {
  const base = new Date((isoDate || '2026-10-01') + 'T00:00:00');
  base.setDate(base.getDate() + Number(days || 30));
  return base.toISOString().slice(0, 10);
}

function inferirEmpresaId(cat) {
  if (cat === 'AC') return 2;
  if (cat === 'ELEVADOR' || cat === 'ESCADAS ROLANTES') return 3;
  return 1;
}

function buildInitialEquipamentos() {
  return EQUIPAMENTOS_BASE.map(item => {
    const id = item[0], nome = item[1], cat = item[2], simbolo = item[3], obsCod = item[4];
    return {
      id: id,
      codigo: String(id),
      nome: nome,
      cat: cat,
      empresaId: inferirEmpresaId(cat),
      estado: simbolo === 'X' ? 'INOP' : 'OP',
      obsCod: obsCod,
      obs: obsCod || '',
      cicloDias: 30,
      ultimaPrev: '2026-10-01',
      proxPrev: addDaysISO('2026-10-01', 30)
    };
  });
}

function buildInitialMapa(equipamentos) {
  const m = {};
  equipamentos.forEach(eq => { m[eq.id] = Array(DIAS).fill(eq.estado === 'INOP' ? 'X' : '='); });
  return m;
}

const INITIAL_DATA = {
  sessaoAtualId: 1,
  cloudConfig: {
    provider: 'firebase',
    url: DEFAULT_FIREBASE_URL,
    path: DEFAULT_FIREBASE_PATH,
    lastSync: ''
  },
  configRelatorio: {
    mes: 'OUTUBRO',
    ano: '2026',
    vistoData: '    /     /',
    vistoCargo: 'O DIRECTOR',
    logoDataUrl: ''
  },
  empresas: [
    { id: 1, nome: 'AIM — Manutenção Mecânica e Transportes', tipo: 'Interna', area: 'Viaturas, SLCI, Limpeza, Pontes, Tapetes, Chillers & UTAs', contacto: 'Aeroporto Internacional de Maputo' },
    { id: 2, nome: 'SeeChanges Electrical & Service, Lda.', tipo: 'Subcontratada', area: "Manutenção Preventiva dos AC's no AIM", contacto: 'NUIT 4008765509' },
    { id: 3, nome: 'SEC África Co., Lda.', tipo: 'Subcontratada', area: 'Elevadores e Escadas Rolantes', contacto: 'AIM — Maputo' },
    { id: 4, nome: 'Fornecedora Geral de Peças & Lubrificantes', tipo: 'Fornecedora de Peças', area: 'Peças, óleos, correias e rolamentos', contacto: 'Maputo' }
  ],
  tecnicos: [
    { id: 1, idPortatil: gerarIdPortatil('Delpódio Eugénio Balate', '849122'), nome: 'Delpódio Eugénio Balate', cargo: 'Estagiário', esp: 'Engenharia Mecânica & Manutenção', pin: '849122' }
  ],
  equipamentos: buildInitialEquipamentos(),
  intervencoes: [],
  pedidosPecas: [],
  propostasPagamentos: [],
  mapa: {}
};
INITIAL_DATA.mapa = buildInitialMapa(INITIAL_DATA.equipamentos);

let state = loadState();
let fotosPendentes = [];
let participantesAtividade = [];
let isSyncingNow = false;

function loadState() {
  try {
    const raw = localStorage.getItem(STORAGE_KEY);
    if (raw) {
      const parsed = JSON.parse(raw);
      if (parsed && Array.isArray(parsed.equipamentos)) {
        if (!parsed.cloudConfig || !parsed.cloudConfig.url) {
          parsed.cloudConfig = { provider: 'firebase', url: DEFAULT_FIREBASE_URL, path: DEFAULT_FIREBASE_PATH, lastSync: '' };
        }
        return parsed;
      }
    }
  } catch (e) {}
  return JSON.parse(JSON.stringify(INITIAL_DATA));
}

function saveState(skipCloud) {
  try { localStorage.setItem(STORAGE_KEY, JSON.stringify(state)); } catch (e) {}
  aplicarConfigRelatorioVisual();
  if (!skipCloud) cloudPushSilencioso();
}

function esc(str) {
  return String(str == null ? '' : str).replace(/[&<>"']/g, m => ({ '&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;' }[m]));
}

function openTab(id, btn) {
  document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
  document.querySelectorAll('#main-nav button').forEach(b => b.classList.remove('active'));
  const target = document.getElementById(id);
  if (target) target.classList.add('active');
  if (btn) btn.classList.add('active');
  window.scrollTo(0, 0);
}

function getTecnicoAtual() {
  return state.tecnicos.find(t => t.id === state.sessaoAtualId) || state.tecnicos[0];
}

function getEmpresaNome(id) {
  const emp = state.empresas.find(e => e.id === Number(id));
  return emp ? emp.nome : 'AIM — Manutenção Mecânica e Transportes';
}

function getCloudConfig() {
  return state.cloudConfig || { provider: 'firebase', url: DEFAULT_FIREBASE_URL, path: DEFAULT_FIREBASE_PATH, lastSync: '' };
}

function salvarConfigNuvem() {
  state.cloudConfig = {
    provider: document.getElementById('cloud-provider').value || 'firebase',
    url: (document.getElementById('cloud-url').value || DEFAULT_FIREBASE_URL).trim().replace(/\/+$/, ''),
    path: (document.getElementById('cloud-path').value || DEFAULT_FIREBASE_PATH).trim().replace(/^\/+|\/+$/g, ''),
    lastSync: (state.cloudConfig && state.cloudConfig.lastSync) || ''
  };
  saveState(true);
  renderCloudUI();
}

function renderCloudUI() {
  const cfg = getCloudConfig();
  const badge = document.getElementById('cloud-status-badge');
  const lastEl = document.getElementById('cloud-last-sync');
  if (document.getElementById('cloud-provider')) document.getElementById('cloud-provider').value = cfg.provider || 'firebase';
  if (document.getElementById('cloud-url')) document.getElementById('cloud-url').value = cfg.url || DEFAULT_FIREBASE_URL;
  if (document.getElementById('cloud-path')) document.getElementById('cloud-path').value = cfg.path || DEFAULT_FIREBASE_PATH;
  if (badge) {
    if (cfg.provider === 'firebase' && cfg.url) {
      badge.textContent = cfg.lastSync ? 'Firebase Ativo' : 'Firebase Ligado';
      badge.className = 'sync-pill';
    } else {
      badge.textContent = 'Modo Local';
      badge.className = 'sync-pill offline';
    }
  }
  if (lastEl) lastEl.textContent = cfg.lastSync ? 'Última sincronização: ' + cfg.lastSync : 'Pronto para sincronizar.';
}

async function cloudPushSilencioso() {
  const cfg = getCloudConfig();
  if (cfg.provider !== 'firebase' || !cfg.url || isSyncingNow) return;
  isSyncingNow = true;
  try {
    const res = await fetch(cfg.url + '/' + cfg.path + '.json', {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(state)
    });
    if (res.ok) {
      cfg.lastSync = new Date().toLocaleTimeString('pt-PT');
      state.cloudConfig = cfg;
      try { localStorage.setItem(STORAGE_KEY, JSON.stringify(state)); } catch (e) {}
      renderCloudUI();
    }
  } catch (e) {}
  finally { isSyncingNow = false; }
}

async function cloudPullSilencioso(showAlert) {
  const cfg = getCloudConfig();
  if (cfg.provider !== 'firebase' || !cfg.url || isSyncingNow) return;
  isSyncingNow = true;
  try {
    const res = await fetch(cfg.url + '/' + cfg.path + '.json');
    if (!res.ok) throw new Error('Erro HTTP ' + res.status);
    const remoteData = await res.json();
    if (remoteData && Array.isArray(remoteData.equipamentos)) {
      const localCmp = JSON.stringify(Object.assign({}, state, { cloudConfig: null, sessaoAtualId: null }));
      const remoteCmp = JSON.stringify(Object.assign({}, remoteData, { cloudConfig: null, sessaoAtualId: null }));
      if (localCmp !== remoteCmp) {
        const meuIdSessao = state.sessaoAtualId;
        remoteData.sessaoAtualId = meuIdSessao;
        remoteData.cloudConfig = Object.assign({}, cfg, { lastSync: new Date().toLocaleTimeString('pt-PT') });
        state = remoteData;
        saveState(true);
        renderAll();
      } else {
        cfg.lastSync = new Date().toLocaleTimeString('pt-PT');
        state.cloudConfig = cfg;
        renderCloudUI();
      }
      if (showAlert) alert('Sincronizado com o Firebase!');
    } else {
      await cloudPushSilencioso();
      if (showAlert) alert('Estado inicial enviado para o Firebase!');
    }
  } catch (err) {
    if (showAlert) alert('Não foi possível ligar ao Firebase: ' + err.message + '. Verifique se abriu pelo link https:// no Chrome.');
  } finally {
    isSyncingNow = false;
  }
}

function cloudSyncNow() { cloudPullSilencioso(true); }
function cloudPush() { cloudPushSilencioso().then(() => alert('Dados enviados para o Firebase!')); }
function cloudPull() { cloudPullSilencioso(true); }

function cadastrarOuEntrarComCodigo() {
  const nome = (document.getElementById('acesso-nome').value || '').trim();
  const cargo = (document.getElementById('acesso-cargo').value || 'Estagiário').trim();
  const codigo = (document.getElementById('acesso-codigo').value || '').trim();
  if (!nome || !codigo) return alert('Preencha o Nome e o Código de Acesso.');

  const idPortatil = gerarIdPortatil(nome, codigo);
  let existente = state.tecnicos.find(t => t.idPortatil === idPortatil || (String(t.pin).trim() === codigo && String(t.nome).trim().toLowerCase() === nome.toLowerCase()));
  if (existente) {
    existente.nome = nome;
    existente.cargo = cargo;
    existente.pin = codigo;
    state.sessaoAtualId = existente.id;
  } else { e técnica',
    relatorio: relatorio || resumo,
    pecas: pecas,
    fotos: [...fotosPendentes]
  });

  document.getElementById('int-cod').value = '';
  document.getElementById('int-resumo').value = '';
  document.getElementById('int-relatorio').value = '';
  document.getElementById('int-pecas').value = '';
  document.getElementById('int-outro-nome').value = '';
  document.getElementById('int-fotos').value = '';
  document.getElementById('int-fotos-preview').innerHTML = '';
  fotosPendentes = [];
  participantesAtividade = [];

  saveState();
  renderAll();
  alert('Atividade diária registada e sincronizada!');
}

function renderDashboard() {
  const total = state.equipamentos.length;
  const inop = state.equipamentos.filter(e => e.estado === 'INOP').length;
  const op = total - inop;
  const disp = total ? ((op / total) * 100).toFixed(1) : '0.0';
  const atrasadas = state.equipamentos.filter(e => statusPrazo(e.proxPrev).code === 'ATRASADA').length;

  document.getElementById('kpi-cards').innerHTML =
    '<div class="card"><div class="kpi-title">Equipamentos</div><div class="kpi-val">' + total + '</div><small>Operativos: <b>' + op + '</b> (' + disp + '%)</small></div>' +
    '<div class="card"><div class="kpi-title">Inoperativos (X)</div><div class="kpi-val" style="color:var(--err)">' + inop + '</div><small>Requerem atenção</small></div>' +
    '<div class="card"><div class="kpi-title">Preventivas Atrasadas</div><div class="kpi-val" style="color:var(--warn)">' + atrasadas + '</div><small>Ver aba Preventivas</small></div>' +
    '<div class="card"><div class="kpi-title">Atividades Registadas</div><div class="kpi-val" style="color:var(--accent)">' + state.intervencoes.length + '</div><small>Pedidos Peças: <b>' + state.pedidosPecas.length + '</b></small></div>';

  const alertas = state.equipamentos.filter(e => e.estado === 'INOP' || statusPrazo(e.proxPrev).code !== 'EM_DIA').slice(0, 20);
  document.getElementById('dash-alerts').innerHTML = alertas.length ? alertas.map(e => {
    const st = statusPrazo(e.proxPrev);
    return '<tr><td><b>' + esc(e.codigo) + '</b></td><td><b>' + esc(e.nome) + '</b></td><td>' + esc(getEmpresaNome(e.empresaId)) + '</td><td><b>' + esc(e.proxPrev || '—') + '</b></td><td>' +
      (e.estado === 'INOP' ? '<span class="badge b-err">INOP (X)</span> ' : '') + '<span class="badge ' + st.cls + '">' + st.label + '</span></td><td>' + esc(e.obs || e.obsCod || '—') + '</td></tr>';
  }).join('') : '<tr><td colspan="6">Sem alertas críticos.</td></tr>';

  document.getElementById('dash-recentes').innerHTML = state.intervencoes.slice(0, 6).map(i => {
    const respNome = i.tecnicoResponsavelNome || '—';
    const respCargo = i.tecnicoResponsavelCargo || '';
    const extras = (i.outrosTecnicos && i.outrosTecnicos.length) ? '<br><small><b>Com:</b> ' + esc(i.outrosTecnicos.join(', ')) + '</small>' : '';
    const eq = state.equipamentos.find(e => e.id === i.eqId);
    return '<tr><td>' + esc(i.data) + '</td><td><b>' + esc(respNome) + '</b> <small>(' + esc(respCargo) + ')</small>' + extras + '</td><td>' + esc(eq ? eq.nome : '—') + '</td><td>' + esc(getEmpresaNome(i.empresaId)) + '</td><td><span class="badge ' + (i.tipo.includes('Inoperativo') ? 'b-err' : 'b-ok') + '">' + esc(i.tipo) + '</span></td><td><b>' + esc(i.resumo) + '</b><br><small>' + esc(i.relatorio) + '</small></td><td>' + renderMiniFotos(i.fotos) + '</td></tr>';
  }).join('') || '<tr><td colspan="7">Sem atividades registadas ainda.</td></tr>';
}

function renderIntervencoes() {
  const q = (document.getElementById('filtro-int').value || '').toLowerCase().trim();
  const list = state.intervencoes.filter(i => {
    if (!q) return true;
    const eq = state.equipamentos.find(e => e.id === i.eqId);
    const txt = (i.tecnicoResponsavelNome || '') + ' ' + (i.outrosTecnicos || []).join(' ') + ' ' + (eq ? eq.nome : '') + ' ' + getEmpresaNome(i.empresaId) + ' ' + i.resumo + ' ' + i.relatorio;
    return txt.toLowerCase().includes(q);
  });
  document.getElementById('lista-intervencoes').innerHTML = list.map(i => {
    const eq = state.equipamentos.find(e => e.id === i.eqId);
    const extras = (i.outrosTecnicos && i.outrosTecnicos.length) ? '<br><small><b>Com:</b> ' + esc(i.outrosTecnicos.join(', ')) + '</small>' : '';
    return '<tr><td>' + esc(i.data) + '</td><td><b>' + esc(i.tecnicoResponsavelNome || '—') + '</b>' + extras + '</td><td><b>' + esc(eq ? eq.nome : '—') + '</b></td><td>' + esc(getEmpresaNome(i.empresaId)) + '</td><td><span class="badge ' + (i.tipo.includes('Inoperativo') ? 'b-err' : 'b-ok') + '">' + esc(i.tipo) + '</span><br><small>Cód: ' + esc(i.cod || '—') + '</small></td><td><b>' + esc(i.resumo) + '</b><br>' + esc(i.relatorio) + '</td><td>' + esc(i.pecas) + '</td><td>' + renderMiniFotos(i.fotos) + '</td></tr>';
  }).join('') || '<tr><td colspan="8">Nenhuma atividade encontrada.</td></tr>';
}

function renderPreventivas() {
  const empFiltro = Number(document.getElementById('prev-filter-emp').value || 0);
  const stFiltro = document.getElementById('prev-filter-status').value;
  const empresasVisiveis = empFiltro ? state.empresas.filter(e => e.id === empFiltro) : state.empresas;
  document.getElementById('preventivas-por-empresa').innerHTML = empresasVisiveis.map(emp => {
    const eqs = state.equipamentos.filter(eq => Number(eq.empresaId) === emp.id && (!stFiltro || statusPrazo(eq.proxPrev).code === stFiltro));
    if (!eqs.length && empFiltro !== emp.id) return '';
    const rows = eqs.length ? eqs.map(eq => {
      const st = statusPrazo(eq.proxPrev);
      return '<tr><td><b>' + esc(eq.codigo) + '</b></td><td><b>' + esc(eq.nome) + '</b><br><small>' + esc(eq.cat) + '</small></td><td>' + (eq.cicloDias || 30) + 'd</td><td>' + esc(eq.ultimaPrev || '—') + '</td><td><b>' + esc(eq.proxPrev || '—') + '</b></td><td><span class="badge ' + st.cls + '">' + st.label + '</span></td><td>' + esc(eq.obs || '—') + '</td><td class="no-print"><button class="btn btn-sm btn-ok" type="button" onclick="concluirPreventivaAgora(' + eq.id + ')">Concluir</button> <button class="btn btn-sm" type="button" onclick="abrirEditarPreventiva(' + eq.id + ')">Editar</button></td></tr>';
    }).join('') : '<tr><td colspan="8">Sem equipamentos neste filtro.</td></tr>';
    return '<div class="card"><h3 style="margin-bottom:8px;color:var(--primary);">' + esc(emp.nome) + '</h3><div style="overflow-x:auto;"><table><thead><tr><th>Item</th><th>Equipamento</th><th>Ciclo</th><th>Última</th><th>Próximo Dia</th><th>Prazo</th><th>Obs.</th><th class="no-print">Ações</th></tr></thead><tbody>' + rows + '</tbody></table></div></div>';
  }).join('');
}

function concluirPreventivaAgora(eqId) {
  const eq = state.equipamentos.find(e => e.id === eqId);
  if (!eq) return;
  const hoje = '2026-11-22';
  eq.ultimaPrev = hoje;
  eq.proxPrev = addDaysISO(hoje, eq.cicloDias || 30);
  eq.obs = 'Concluída em ' + hoje;
  saveState();
  renderAll();
}

function abrirEditarPreventiva(eqId) {
  const eq = state.equipamentos.find(e => e.id === eqId);
  if (!eq) return;
  document.getElementById('edit-prev-id').value = eq.id;
  document.getElementById('edit-prev-nome').value = eq.codigo + ' — ' + eq.nome;
  document.getElementById('edit-prev-emp').value = String(eq.empresaId || 1);
  document.getElementById('edit-prev-ciclo').value = eq.cicloDias || 30;
  document.getElementById('edit-prev-ultima').value = eq.ultimaPrev || '2026-10-01';
  document.getElementById('edit-prev-prox').value = eq.proxPrev || addDaysISO('2026-10-01', 30);
  document.getElementById('edit-prev-obs').value = eq.obs || '';
  document.getElementById('edit-prev-box').style.display = 'block';
}

function recalcularProxPrevInput() {
  const ult = document.getElementById('edit-prev-ultima').value;
  const ciclo = Number(document.getElementById('edit-prev-ciclo').value || 30);
  if (ult) document.getElementById('edit-prev-prox').value = addDaysISO(ult, ciclo);
}

function salvarEdicaoPreventiva() {
  const eq = state.equipamentos.find(e => e.id === Number(document.getElementById('edit-prev-id').value));
  if (!eq) return;
  eq.empresaId = Number(document.getElementById('edit-prev-emp').value);
  eq.cicloDias = Math.max(1, Number(document.getElementById('edit-prev-ciclo').value || 30));
  eq.ultimaPrev = document.getElementById('edit-prev-ultima').value;
  eq.proxPrev = document.getElementById('edit-prev-prox').value;
  eq.obs = document.getElementById('edit-prev-obs').value.trim();
  document.getElementById('edit-prev-box').style.display = 'none';
  saveState();
  renderAll();
}

function addEquipamento() {
  const nome = document.getElementById('eq-nome').value.trim().toUpperCase();
  if (!nome) return alert('Indique o nome do equipamento.');
  const nextId = state.equipamentos.length ? Math.max(...state.equipamentos.map(e => e.id)) + 1 : 1;
  const ciclo = Math.max(1, Number(document.getElementById('eq-ciclo').value || 30));
  const ult = document.getElementById('eq-ultima').value || '2026-10-01';
  state.equipamentos.push({
    id: nextId,
    codigo: document.getElementById('eq-codigo').value.trim() || String(nextId),
    nome: nome,
    cat: document.getElementById('eq-cat').value,
    empresaId: Number(document.getElementById('eq-emp').value),
    estado: 'OP',
    obsCod: document.getElementById('eq-obs-cod').value.trim(),
    obs: document.getElementById('eq-obs').value.trim(),
    cicloDias: ciclo,
    ultimaPrev: ult,
    proxPrev: addDaysISO(ult, ciclo)
  });
  state.mapa[nextId] = Array(DIAS).fill('=');
  document.getElementById('eq-nome').value = '';
  saveState();
  renderAll();
}

function abrirEditarEquipamento(id) {
  const eq = state.equipamentos.find(e => e.id === id);
  if (!eq) return;
  document.getElementById('edit-eq-id').value = eq.id;
  document.getElementById('edit-eq-codigo').value = eq.codigo;
  document.getElementById('edit-eq-nome').value = eq.nome;
  document.getElementById('edit-eq-cat').value = eq.cat;
  document.getElementById('edit-eq-emp').value = String(eq.empresaId || 1);
  document.getElementById('edit-eq-estado').value = eq.estado;
  document.getElementById('edit-eq-obscod').value = eq.obsCod || '';
  document.getElementById('edit-eq-obs').value = eq.obs || '';
  document.getElementById('edit-eq-box').style.display = 'block';
}

function salvarEdicaoEquipamento() {
  const eq = state.equipamentos.find(e => e.id === Number(document.getElementById('edit-eq-id').value));
  if (!eq) return;
  eq.codigo = document.getElementById('edit-eq-codigo').value.trim() || eq.codigo;
  eq.nome = document.getElementById('edit-eq-nome').value.trim().toUpperCase() || eq.nome;
  eq.cat = document.getElementById('edit-eq-cat').value;
  eq.empresaId = Number(document.getElementById('edit-eq-emp').value);
  eq.estado = document.getElementById('edit-eq-estado').value;
  eq.obsCod = document.getElementById('edit-eq-obscod').value.trim();
  eq.obs = document.getElementById('edit-eq-obs').value.trim();
  document.getElementById('edit-eq-box').style.display = 'none';
  saveState();
  renderAll();
}

function renderEquipamentos() {
  const q = (document.getElementById('search-eq').value || '').toLowerCase().trim();
  const cat = document.getElementById('filter-cat').value;
  const empId = Number(document.getElementById('filter-emp').value || 0);
  const list = state.equipamentos.filter(e => (!q || e.nome.toLowerCase().includes(q) || e.codigo.toLowerCase().includes(q)) && (!cat || e.cat === cat) && (!empId || Number(e.empresaId) === empId));
  document.getElementById('lista-equipamentos').innerHTML = list.map(e =>
    '<tr><td><b>' + esc(e.codigo) + '</b></td><td><b>' + esc(e.nome) + '</b></td><td>' + esc(e.cat) + '</td><td>' + esc(getEmpresaNome(e.empresaId)) + '</td><td><span class="badge ' + (e.estado === 'OP' ? 'b-ok' : 'b-err') + '">' + (e.estado === 'OP' ? 'OPERATIVO (=)' : 'INOPERATIVO (X)') + '</span></td><td>' + esc(e.proxPrev || '—') + '</td><td>' + esc(e.obs || e.obsCod || '—') + '</td><td><button class="btn btn-sm btn-warn" type="button" onclick="abrirEditarEquipamento(' + e.id + ')">Editar</button></td></tr>'
  ).join('');
}

function addEmpresa() {
  const nome = document.getElementById('sub-nome').value.trim();
  if (!nome) return alert('Indique o nome da empresa.');
  const nextId = state.empresas.length ? Math.max(...state.empresas.map(e => e.id)) + 1 : 1;
  state.empresas.push({ id: nextId, nome: nome, tipo: document.getElementById('sub-tipo').value, area: document.getElementById('sub-area').value.trim() || 'Serviços', contacto: document.getElementById('sub-contacto').value.trim() || '—' });
  document.getElementById('sub-nome').value = '';
  saveState();
  renderAll();
}

function renderEmpresas() {
  document.getElementById('lista-empresas').innerHTML = state.empresas.map(emp => {
    const qtd = state.equipamentos.filter(eq => Number(eq.empresaId) === emp.id).length;
    return '<tr><td><b>' + esc(emp.nome) + '</b></td><td><span class="badge b-info">' + esc(emp.tipo) + '</span></td><td>' + esc(emp.area) + '</td><td>' + esc(emp.contacto) + '</td><td><b>' + qtd + '</b></td><td><button class="btn btn-sm" type="button" onclick="document.getElementById(\'filter-emp\').value=\'' + emp.id + '\';openTab(\'equipamentos\');renderEquipamentos();">Ver Equipamentos</button></td></tr>';
  }).join('');
}

function addPedidoPeca() {
  const peca = document.getElementById('ped-peca').value.trim();
  if (!peca) return alert('Indique a peça.');
  state.pedidosPecas.unshift({ id: Date.now(), data: '2026-11-22', eqId: Number(document.getElementById('ped-eq').value), peca: peca, qtd: document.getElementById('ped-qtd').value || '1', fornecedorId: Number(document.getElementById('ped-forn').value), status: document.getElementById('ped-status').value });
  document.getElementById('ped-peca').value = '';
  saveState();
  renderAll();
}

function avancarStatusPedido(id) {
  const ordem = ['Solicitado', 'Aguarda Cotação', 'Aprovado', 'Entregue'];
  const p = state.pedidosPecas.find(x => x.id === id);
  if (!p) return;
  p.status = ordem[(ordem.indexOf(p.status) + 1) % ordem.length];
  saveState();
  renderAll();
}

function renderPedidosPecas() {
  document.getElementById('lista-pedidos-pecas').innerHTML = state.pedidosPecas.map(p => {
    const eq = state.equipamentos.find(e => e.id === p.eqId);
    return '<tr><td>' + esc(p.data) + '</td><td><b>' + esc(eq ? eq.nome : '—') + '</b></td><td>' + esc(p.peca) + '</td><td><b>' + esc(p.qtd) + '</b></td><td>' + esc(getEmpresaNome(p.fornecedorId)) + '</td><td><span class="badge b-warn">' + esc(p.status) + '</span></td><td><button class="btn btn-sm" type="button" onclick="avancarStatusPedido(' + p.id + ')">Mudar Estado</button></td></tr>';
  }).join('') || '<tr><td colspan="7">Sem pedidos de peças.</td></tr>';
}

function addPropostaPagamento() {
  const desc = document.getElementById('prop-desc').value.trim();
  if (!desc) return alert('Descreva a proposta.');
  state.propostasPagamentos.unshift({ id: Date.now(), data: document.getElementById('prop-data').value || '2026-11-22', empresaId: Number(document.getElementById('prop-emp').value), desc: desc, valor: document.getElementById('prop-valor').value.trim() || 'A definir', status: document.getElementById('prop-status').value });
  document.getElementById('prop-desc').value = '';
  saveState();
  renderAll();
}

function avancarStatusProposta(id) {
  const ordem = ['Proposta Recebida', 'Aprovada (Aguarda Pagamento)', 'Paga', 'Rejeitada'];
  const p = state.propostasPagamentos.find(x => x.id === id);
  if (!p) return;
  p.status = ordem[(ordem.indexOf(p.status) + 1) % ordem.length];
  saveState();
  renderAll();
}

function renderPropostasPagamentos() {
  document.getElementById('lista-propostas').innerHTML = state.propostasPagamentos.map(p =>
    '<tr><td>' + esc(p.data) + '</td><td><b>' + esc(getEmpresaNome(p.empresaId)) + '</b></td><td>' + esc(p.desc) + '</td><td><b>' + esc(p.valor) + '</b></td><td><span class="badge b-info">' + esc(p.status) + '</span></td><td><button class="btn btn-sm" type="button" onclick="avancarStatusProposta(' + p.id + ')">Atualizar</button></td></tr>'
  ).join('') || '<tr><td colspan="6">Sem propostas ou pagamentos.</td></tr>';
}

function addMembro() {
  const nome = document.getElementById('tec-nome').value.trim();
  if (!nome) return alert('Indique o nome.');
  const pin = document.getElementById('tec-pin').value.trim() || '123456';
  const nextId = state.tecnicos.length ? Math.max(...state.tecnicos.map(t => t.id)) + 1 : 1;
  state.tecnicos.push({ id: nextId, idPortatil: gerarIdPortatil(nome, pin), nome: nome, cargo: document.getElementById('tec-cargo').value, esp: document.getElementById('tec-esp').value.trim() || 'Manutenção', pin: pin });
  document.getElementById('tec-nome').value = '';
  document.getElementById('tec-esp').value = '';
  document.getElementById('tec-pin').value = '';
  saveState();
  renderAll();
}

function removerMembro(id) {
  if (state.tecnicos.length <= 1) return alert('Mantenha pelo menos 1 técnico.');
  state.tecnicos = state.tecnicos.filter(t => t.id !== id);
  if (state.sessaoAtualId === id) state.sessaoAtualId = state.tecnicos[0].id;
  saveState();
  renderAll();
}

function renderTecnicos() {
  document.getElementById('lista-tecnicos').innerHTML = state.tecnicos.map(t =>
    '<tr><td><code>' + esc(t.idPortatil || gerarIdPortatil(t.nome, t.pin)) + '</code></td><td><b>' + esc(t.nome) + '</b></td><td><span class="badge b-info">' + esc(t.cargo) + '</span></td><td>' + esc(t.esp) + '</td><td><b>' + esc(t.pin) + '</b></td><td><button class="btn btn-sm btn-err" type="button" onclick="removerMembro(' + t.id + ')">Remover</button></td></tr>'
  ).join('');
}

function salvarConfigRelatorio() {
  state.configRelatorio.mes = (document.getElementById('rel-mes').value || 'OUTUBRO').toUpperCase();
  state.configRelatorio.ano = document.getElementById('rel-ano').value || '2026';
  state.configRelatorio.vistoData = document.getElementById('rel-visto-data').value || '    /     /';
  state.configRelatorio.vistoCargo = (document.getElementById('rel-visto-cargo').value || 'O DIRECTOR').toUpperCase();
  saveState();
}

function salvarLogoAIM() {
  const file = document.getElementById('logo-aim-file').files[0];
  if (!file) return;
  const reader = new FileReader();
  reader.onload = ev => { state.configRelatorio.logoDataUrl = ev.target.result; saveState(); };
  reader.readAsDataURL(file);
}

function aplicarConfigRelatorioVisual() {
  const cfg = state.configRelatorio || {};
  const wm = document.getElementById('print-watermark');
  if (wm) wm.innerHTML = cfg.logoDataUrl ? '<img src="' + cfg.logoDataUrl + '" alt="AIM">' : '';
  if (document.getElementById('sheet-mes')) document.getElementById('sheet-mes').textContent = (cfg.mes || 'OUTUBRO').toUpperCase();
  if (document.getElementById('sheet-ano')) document.getElementById('sheet-ano').textContent = cfg.ano || '2026';
  if (document.getElementById('sheet-visto-data')) document.getElementById('sheet-visto-data').textContent = cfg.vistoData || '    /     /';
  if (document.getElementById('sheet-visto-cargo')) document.getElementById('sheet-visto-cargo').textContent = (cfg.vistoCargo || 'O DIRECTOR').toUpperCase();
}

function renderMapaMensal() {
  const empId = Number(document.getElementById('rep-emp').value || 0);
  const eqs = empId ? state.equipamentos.filter(e => Number(e.empresaId) === empId) : state.equipamentos;
  const diasH = Array.from({ length: DIAS }, (_, i) => '<th>' + (i + 1) + '</th>').join('');
  let corpo = '';
  CATEGORIAS.forEach(cat => {
    const grupo = eqs.filter(e => e.cat === cat);
    if (!grupo.length) return;
    corpo += '<tr class="ex-cat-row"><td colspan="' + (DIAS + 3) + '">' + esc(cat) + '</td></tr>';
    grupo.forEach(eq => {
      if (!state.mapa[eq.id]) state.mapa[eq.id] = Array(DIAS).fill(eq.estado === 'INOP' ? 'X' : '=');
      const cells = state.mapa[eq.id].map((val, dIdx) =>
        '<td class="col-dia ' + (val === 'X' ? 'cell-inop' : 'cell-op') + '" onclick="toggleCell(' + eq.id + ',' + dIdx + ')">' + val + '</td>'
      ).join('');
      corpo += '<tr><td class="col-item">' + esc(eq.codigo) + '</td><td class="col-eq">' + esc(eq.nome) + '</td>' + cells + '<td class="col-obs">' + esc(eq.obsCod || eq.obs || '') + '</td></tr>';
    });
  });
  document.getElementById('mapa-mensal').innerHTML = '<thead><tr><th>ITEM</th><th>TIPO DE EQUIPAMENTO</th>' + diasH + '<th>OBSERVAÇÕES</th></tr></thead><tbody>' + corpo + '</tbody>';
}

function toggleCell(eqId, dIdx) {
  if (!state.mapa[eqId]) state.mapa[eqId] = Array(DIAS).fill('=');
  state.mapa[eqId][dIdx] = state.mapa[eqId][dIdx] === '=' ? 'X' : '=';
  const eq = state.equipamentos.find(e => e.id === eqId);
  if (eq) eq.estado = state.mapa[eqId][state.mapa[eqId].length - 1] === 'X' ? 'INOP' : 'OP';
  saveState();
  renderAll();
}

function baixarFicheiro(nome, conteudo, mime) {
  const blob = new Blob(['\ufeff' + conteudo], { type: mime });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = nome;
  document.body.appendChild(a);
  a.click();
  a.remove();
  URL.revokeObjectURL(url);
}

function exportarWordSemanal() {
  const cfg = state.configRelatorio || {};
  const dIni = document.getElementById('sem-data-inicio').value || '2026-11-16';
  const dFim = document.getElementById('sem-data-fim').value || '2026-11-22';
  const titulo = (document.getElementById('sem-titulo').value || 'RELATÓRIO SEMANAL DE MANUTENÇÃO').trim();
  const empId = Number(document.getElementById('rep-emp').value || 0);
  const tecId = Number(document.getElementById('rep-tec').value || 0);

  const interv = state.intervencoes.filter(i => (!dIni || i.data >= dIni) && (!dFim || i.data <= dFim) && (!empId || Number(i.empresaId) === empId) && (!tecId || Number(i.tecnicoId) === tecId));
  const inopEqs = (empId ? state.equipamentos.filter(e => Number(e.empresaId) === empId) : state.equipamentos).filter(e => e.estado === 'INOP');

  const rowsAtiv = interv.length ? interv.map(i => {
    const eq = state.equipamentos.find(e => e.id === i.eqId);
    const equipa = [i.tecnicoResponsavelNome || '—'].concat(i.outrosTecnicos || []).join(', ');
    return '<tr><td>' + esc(i.data) + '</td><td><b>' + esc(eq ? eq.codigo + ' — ' + eq.nome : '—') + '</b></td><td>' + esc(getEmpresaNome(i.empresaId)) + '</td><td><b>' + esc(i.tipo) + '</b></td><td>' + esc(equipa) + '</td><td><b>' + esc(i.resumo) + '</b><br>' + esc(i.relatorio) + '</td><td>' + esc(i.pecas || 'Nenhuma') + '</td></tr>';
  }).join('') : '<tr><td colspan="7" align="center">Sem atividades registadas neste período.</td></tr>';

  const rowsInop = inopEqs.length ? inopEqs.map(e =>
    '<tr><td align="center"><b>' + esc(e.codigo) + '</b></td><td><b>' + esc(e.nome) + '</b></td><td>' + esc(e.cat) + '</td><td>' + esc(getEmpresaNome(e.empresaId)) + '</td><td>' + esc(e.obsCod || e.obs || 'Inoperativo') + '</td></tr>'
  ).join('') : '<tr><td colspan="5" align="center">Nenhum equipamento inoperativo.</td></tr>';

  const doc = '<html xmlns:o="urn:schemas-microsoft-com:office:office" xmlns:w="urn:schemas-microsoft-com:office:word"><head><meta charset="utf-8"><title>Relatório Semanal</title></head><body style="font-family:Arial,sans-serif;font-size:10px;">' +
    '<div><b>PELOURO DE ENGENHARIA E MANUTENÇÃO</b><br><b>DIRECÇÃO DE MANUTENÇÃO DE EQUIPAMENTOS E SISTEMAS</b><br><b>AEROPORTO INTERNACIONAL DE MAPUTO</b><br><b>MANUTENÇÃO MECÂNICA E TRANSPORTES</b></div>' +
    '<div style="text-align:center;font-weight:bold;font-size:13px;margin:8px 0;">' + esc(titulo.toUpperCase()) + '<br><span style="font-size:11px;font-weight:normal;">Período: <b>' + esc(dIni) + '</b> a <b>' + esc(dFim) + '</b></span></div>' +
    '<p><b>VISTO:</b> ' + esc(cfg.vistoData || '    /     /') + ' — ' + esc((cfg.vistoCargo || 'O DIRECTOR').toUpperCase()) + '</p>' +
    '<h4>1. ATIVIDADES DIÁRIAS EXECUTADAS</h4><table border="1" cellpadding="4" cellspacing="0" style="border-collapse:collapse;width:100%;font-size:9px;"><tr><th>Data</th><th>Equipamento</th><th>Empresa</th><th>Tipo</th><th>Técnicos</th><th>Descrição / Diagnóstico</th><th>Peças</th></tr>' + rowsAtiv + '</table>' +
    '<h4>2. EQUIPAMENTOS INOPERATIVOS (X)</h4><table border="1" cellpadding="4" cellspacing="0" style="border-collapse:collapse;width:100%;font-size:9px;"><tr><th>Item</th><th>Equipamento</th><th>Categoria</th><th>Empresa</th><th>Observação</th></tr>' + rowsInop + '</table></body></html>';

  baixarFicheiro('RELATORIO_SEMANAL_' + dIni + '_a_' + dFim + '.doc', doc, 'application/msword;charset=utf-8;');
}

function exportarWord() {
  const cfg = state.configRelatorio || {};
  const empId = Number(document.getElementById('rep-emp').value || 0);
  const eqs = empId ? state.equipamentos.filter(e => Number(e.empresaId) === empId) : state.equipamentos;
  const thDias = Array.from({ length: DIAS }, (_, i) => '<th>' + (i + 1) + '</th>').join('');
  let rowsHtml = '';
  CATEGORIAS.forEach(cat => {
    const grupo = eqs.filter(e => e.cat === cat);
    if (!grupo.length) return;
    rowsHtml += '<tr><td colspan="' + (DIAS + 3) + '" style="background:#e2e8f0;font-weight:bold;">' + esc(cat) + '</td></tr>';
    grupo.forEach(eq => {
      const dias = state.mapa[eq.id] || Array(DIAS).fill('=');
      rowsHtml += '<tr><td align="center"><b>' + esc(eq.codigo) + '</b></td><td>' + esc(eq.nome) + '</td>' + dias.map(d => '<td align="center">' + d + '</td>').join('') + '<td>' + esc(eq.obsCod || eq.obs || '') + '</td></tr>';
    });
  });
  const docHtml = '<html xmlns:o="urn:schemas-microsoft-com:office:office" xmlns:w="urn:schemas-microsoft-com:office:word"><head><meta charset="utf-8"></head><body style="font-family:Arial,sans-serif;font-size:10px;"><div style="text-align:center;font-weight:bold;font-size:13px;">RELATÓRIO MENSAL — ' + esc((cfg.mes || 'OUTUBRO').toUpperCase()) + ' ' + esc(cfg.ano || '2026') + ' (TABELA Nº 1)</div><table border="1" cellpadding="3" cellspacing="0" style="border-collapse:collapse;width:100%;font-size:9px;"><tr><th>ITEM</th><th>TIPO DE EQUIPAMENTO</th>' + thDias + '<th>OBSERVAÇÕES</th></tr>' + rowsHtml + '</table></body></html>';
  baixarFicheiro('RELATORIO_MENSAL_TABELA_1.doc', docHtml, 'application/msword;charset=utf-8;');
}

function exportarExcel() {
  const header = ['ITEM', 'TIPO DE EQUIPAMENTO', 'CATEGORIA'].concat(Array.from({ length: DIAS }, (_, i) => String(i + 1))).concat(['OBSERVAÇÕES']);
  const linhas = [header.join(';')];
  state.equipamentos.forEach(eq => {
    const dias = state.mapa[eq.id] || Array(DIAS).fill('=');
    linhas.push([eq.codigo, eq.nome, eq.cat].concat(dias).concat([eq.obsCod || eq.obs || '']).join(';'));
  });
  baixarFicheiro('RELATORIO_MENSAL_TABELA_1.csv', linhas.join('\n'), 'text/csv;charset=utf-8;');
}

function exportarBackupJSON() {
  baixarFicheiro('SGM-AIM_Backup.json', JSON.stringify(state, null, 2), 'application/json;charset=utf-8;');
}

function importarBackupJSON() {
  const file = document.getElementById('import-json-file').files[0];
  if (!file) return;
  const reader = new FileReader();
  reader.onload = ev => {
    try {
      const parsed = JSON.parse(ev.target.result);
      if (parsed && Array.isArray(parsed.equipamentos)) {
        parsed.cloudConfig = getCloudConfig();
        state = parsed;
        saveState();
        renderAll();
        alert('Backup restaurado!');
      }
    } catch (e) { alert('JSON inválido.'); }
  };
  reader.readAsText(file);
}

function renderAll() {
  if (window.location.protocol === 'file:') {
    const al = document.getElementById('alerta-ficheiro-local');
    if (al) al.style.display = 'block';
  }
  renderHeaderUser();
  renderCloudUI();
  populateSelects();
  renderParticipantesAtividade();
  renderDashboard();
  renderIntervencoes();
  renderPreventivas();
  renderEquipamentos();
  renderPedidosPecas();
  renderEmpresas();
  renderTecnicos();
  renderMapaMensal();
  aplicarConfigRelatorioVisual();
}

renderAll();
cloudPullSilencioso(false);
setInterval(() => { if (navigator.onLine) cloudPullSilencioso(false); }, 15000);
</script>
</body>
</html>
