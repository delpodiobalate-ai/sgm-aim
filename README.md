



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
  } else {
