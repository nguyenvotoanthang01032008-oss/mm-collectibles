# mm-collectibles
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width,initial-scale=1" />
    <title>Client Tracker (Offline)</title>
    <style>
      :root {
        --bg: #0b1020;
        --panel: rgba(255, 255, 255, 0.06);
        --panel2: rgba(255, 255, 255, 0.09);
        --text: rgba(255, 255, 255, 0.92);
        --muted: rgba(255, 255, 255, 0.68);
        --border: rgba(255, 255, 255, 0.14);
        --border2: rgba(255, 255, 255, 0.22);
        --shadow: 0 14px 40px rgba(0, 0, 0, 0.35);
        --radius: 14px;
        
        --lead: #94a3b8; /* slate */
        --active: #22c55e; /* green */
        --paused: #f59e0b; /* amber */
        --closedWon: #14b8a6; /* teal */
        --closedLost: #ef4444; /* red */
      }

      * { box-sizing: border-box; }
      html, body { height: 100%; }
      body {
        margin: 0;
        color: var(--text);
        background:
          radial-gradient(1000px 500px at 10% 0%, rgba(110, 231, 183, 0.12), transparent 55%),
          radial-gradient(900px 500px at 90% 10%, rgba(96, 165, 250, 0.12), transparent 55%),
          radial-gradient(900px 500px at 60% 90%, rgba(244, 114, 182, 0.10), transparent 55%),
          var(--bg);
        font-family: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial, "Apple Color Emoji", "Segoe UI Emoji";
      }

      a { color: inherit; }
      button, input, select, textarea { font: inherit; }

      .wrap { max-width: 1300px; margin: 24px auto; padding: 0 16px 40px; }
      .topbar {
        display: flex; align-items: center; justify-content: space-between; gap: 14px;
        padding: 14px 16px; border: 1px solid var(--border); border-radius: var(--radius);
        background: linear-gradient(180deg, rgba(255,255,255,0.08), rgba(255,255,255,0.03));
        box-shadow: var(--shadow);
      }
      .brand { display: flex; align-items: baseline; gap: 12px; }
      .brand h1 { font-size: 16px; margin: 0; letter-spacing: 0.2px; }
      .brand .sub { color: var(--muted); font-size: 12px; }

      .actions { display: flex; flex-wrap: wrap; align-items: center; gap: 10px; }
      .btn {
        cursor: pointer;
        border: 1px solid var(--border);
        background: rgba(255, 255, 255, 0.06);
        color: var(--text);
        padding: 9px 10px;
        border-radius: 12px;
        transition: transform 0.05s ease, background 0.15s ease, border-color 0.15s ease;
        user-select: none;
      }
      .btn:hover { background: rgba(255, 255, 255, 0.10); border-color: var(--border2); }
      .btn:active { transform: translateY(1px); }
      .btn.primary {
        border-color: rgba(59, 130, 246, 0.55);
        background: rgba(59, 130, 246, 0.20);
      }
      .btn.danger {
        border-color: rgba(239, 68, 68, 0.50);
        background: rgba(239, 68, 68, 0.16);
      }
      .btn.small { padding: 7px 9px; border-radius: 10px; font-size: 12px; }

      .grid { display: grid; grid-template-columns: 1.4fr 1fr; gap: 14px; margin-top: 14px; }
      @media (max-width: 980px) { .grid { grid-template-columns: 1fr; } }

      .card {
        border: 1px solid var(--border);
        background: rgba(255, 255, 255, 0.05);
        border-radius: var(--radius);
        box-shadow: var(--shadow);
        overflow: hidden;
      }
      .card .hd {
        padding: 12px 14px;
        display: flex; align-items: center; justify-content: space-between; gap: 10px;
        border-bottom: 1px solid var(--border);
        background: rgba(255, 255, 255, 0.05);
      }
      .card .hd .title { font-size: 13px; margin: 0; letter-spacing: 0.2px; color: rgba(255,255,255,0.86); }
      .card .bd { padding: 12px 14px; }

      .dash {
        display: grid;
        grid-template-columns: repeat(5, 1fr);
        gap: 10px;
      }
      @media (max-width: 980px) { .dash { grid-template-columns: repeat(2, 1fr); } }
      .stat {
        border: 1px solid var(--border);
        background: rgba(0,0,0,0.12);
        border-radius: 14px;
        padding: 10px 10px 9px;
        min-height: 64px;
      }
      .stat .k { font-size: 11px; color: var(--muted); margin-bottom: 6px; }
      .stat .v { font-size: 18px; font-weight: 700; letter-spacing: 0.3px; }
      .stat .hint { font-size: 11px; margin-top: 4px; color: var(--muted); }

      .filters {
        display: grid;
        grid-template-columns: 1.2fr 1fr 1fr 1fr 0.9fr;
        gap: 10px;
      }
      @media (max-width: 980px) { .filters { grid-template-columns: 1fr 1fr; } }
      .field label { display: block; font-size: 11px; color: var(--muted); margin: 0 0 6px; }
      .field input, .field select {
        width: 100%;
        padding: 9px 10px;
        border-radius: 12px;
        border: 1px solid var(--border);
        background: rgba(255,255,255,0.06);
        color: var(--text);
        outline: none;
      }
      .field input::placeholder { color: rgba(255,255,255,0.45); }

      table { width: 100%; border-collapse: separate; border-spacing: 0; }
      thead th {
        position: sticky; top: 0;
        z-index: 1;
        background: rgba(12, 18, 38, 0.92);
        backdrop-filter: blur(8px);
        text-align: left;
        font-size: 11px;
        color: rgba(255,255,255,0.72);
        padding: 10px 10px;
        border-bottom: 1px solid var(--border);
        cursor: pointer;
        user-select: none;
        white-space: nowrap;
      }
      thead th.sort { color: rgba(255,255,255,0.92); }
      tbody td {
        font-size: 12.5px;
        padding: 10px 10px;
        border-bottom: 1px solid rgba(255,255,255,0.08);
        vertical-align: top;
      }
      tbody tr:hover { background: rgba(255,255,255,0.04); }

      .mono { font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; }
      .muted { color: var(--muted); }
      .nowrap { white-space: nowrap; }

      .badge {
        display: inline-flex; align-items: center;
        padding: 3px 8px;
        border-radius: 999px;
        font-size: 11px;
        font-weight: 700;
        letter-spacing: 0.2px;
        border: 1px solid rgba(255,255,255,0.14);
        background: rgba(255,255,255,0.06);
      }
      .badge.lead { border-color: rgba(148,163,184,0.45); color: var(--lead); }
      .badge.active { border-color: rgba(34,197,94,0.45); color: var(--active); }
      .badge.paused { border-color: rgba(245,158,11,0.50); color: var(--paused); }
      .badge.closedWon { border-color: rgba(20,184,166,0.50); color: var(--closedWon); }
      .badge.closedLost { border-color: rgba(239,68,68,0.50); color: var(--closedLost); }

      .pill {
        display: inline-flex; align-items: center;
        padding: 3px 8px;
        border-radius: 999px;
        font-size: 11px;
        border: 1px solid rgba(255,255,255,0.14);
        background: rgba(255,255,255,0.04);
        color: rgba(255,255,255,0.78);
        white-space: nowrap;
      }

      .tableWrap {
        max-height: 62vh;
        overflow: auto;
      }

      dialog {
        border: 1px solid var(--border);
        border-radius: 18px;
        background: rgba(12, 18, 38, 0.92);
        color: var(--text);
        box-shadow: var(--shadow);
        width: min(980px, calc(100vw - 28px));
      }
      dialog::backdrop { background: rgba(0,0,0,0.55); }
      .dlgHd { display: flex; align-items: center; justify-content: space-between; gap: 10px; padding: 14px 14px 10px; border-bottom: 1px solid var(--border); }
      .dlgHd h2 { margin: 0; font-size: 13px; letter-spacing: 0.2px; }
      .dlgBd { padding: 14px; }
      .formGrid {
        display: grid;
        grid-template-columns: repeat(3, 1fr);
        gap: 10px;
      }
      @media (max-width: 980px) { .formGrid { grid-template-columns: 1fr; } }
      .formGrid .field textarea {
        width: 100%;
        min-height: 110px;
        resize: vertical;
        padding: 9px 10px;
        border-radius: 12px;
        border: 1px solid var(--border);
        background: rgba(255,255,255,0.06);
        color: var(--text);
        outline: none;
      }
      .dlgFt { display: flex; justify-content: space-between; gap: 10px; padding: 12px 14px 14px; border-top: 1px solid var(--border); }
      .hintline { font-size: 11px; color: var(--muted); margin-top: 8px; line-height: 1.35; }

      .kpiRow { display: flex; flex-wrap: wrap; gap: 8px; align-items: center; }
      .kpiRow .pill strong { font-weight: 800; margin-right: 6px; }
    </style>
  </head>
  <body>
    <div class="wrap">
      <div class="topbar">
        <div class="brand">
          <h1>Client Tracker</h1>
          <div class="sub">Single-file • Offline • Auto-saves in your browser</div>
        </div>
        <div class="actions">
          <button class="btn primary" id="btnAdd">Add client</button>
          <button class="btn" id="btnExport">Export JSON</button>
          <label class="btn" style="display:inline-flex; gap:8px; align-items:center;">
            Import JSON
            <input id="fileImport" type="file" accept="application/json" style="display:none;" />
          </label>
          <button class="btn danger" id="btnReset">Reset local data</button>
        </div>
      </div>

      <div class="grid">
        <div class="card">
          <div class="hd">
            <div class="title">Dashboard</div>
            <div class="kpiRow" id="kpiRow"></div>
          </div>
          <div class="bd">
            <div class="dash" id="dash"></div>
            <div class="hintline">
              Tip: Use “Next action due” filter to surface overdue follow-ups. Export regularly if you want backups you can move between computers.
            </div>
          </div>
        </div>

        <div class="card">
          <div class="hd"><div class="title">Search & filters</div></div>
          <div class="bd">
            <div class="filters">
              <div class="field">
                <label for="q">Search</label>
                <input id="q" placeholder="Name, company, email, notes…" />
              </div>
              <div class="field">
                <label for="status">Status</label>
                <select id="status"></select>
              </div>
              <div class="field">
                <label for="stage">Stage</label>
                <select id="stage"></select>
              </div>
              <div class="field">
                <label for="due">Next action due</label>
                <select id="due">
                  <option value="all">All</option>
                  <option value="overdue">Overdue</option>
                  <option value="7">Next 7 days</option>
                  <option value="30">Next 30 days</option>
                  <option value="none">No due date</option>
                </select>
              </div>
              <div class="field">
                <label for="minValue">Min value ($)</label>
                <input id="minValue" type="number" min="0" step="0.01" placeholder="0" />
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="card" style="margin-top:14px;">
        <div class="hd">
          <div class="title">Clients</div>
          <div class="muted" id="resultMeta" style="font-size:12px;"></div>
        </div>
        <div class="bd" style="padding:0;">
          <div class="tableWrap">
            <table>
              <thead>
                <tr>
                  <th data-sort="clientName">Client</th>
                  <th data-sort="company">Company</th>
                  <th data-sort="status">Status</th>
                  <th data-sort="stage">Stage</th>
                  <th data-sort="dealValueUsd">Value</th>
                  <th data-sort="lastContactDate">Last contact</th>
                  <th data-sort="nextActionDueDate">Next due</th>
                  <th>Next action</th>
                  <th>Contact</th>
                  <th></th>
                </tr>
              </thead>
              <tbody id="rows"></tbody>
            </table>
          </div>
        </div>
      </div>
    </div>

    <dialog id="dlg">
      <div class="dlgHd">
        <h2 id="dlgTitle">Client</h2>
        <button class="btn small" id="btnClose">Close</button>
      </div>
      <div class="dlgBd">
        <div class="formGrid">
          <div class="field">
            <label>Client name</label>
            <input id="f_clientName" />
          </div>
          <div class="field">
            <label>Company</label>
            <input id="f_company" />
          </div>
          <div class="field">
            <label>Status</label>
            <select id="f_status"></select>
          </div>
          <div class="field">
            <label>Stage</label>
            <select id="f_stage"></select>
          </div>
          <div class="field">
            <label>Deal / project value ($)</label>
            <input id="f_dealValueUsd" type="number" min="0" step="0.01" />
          </div>
          <div class="field">
            <label>Start date</label>
            <input id="f_startDate" type="date" />
          </div>
          <div class="field">
            <label>Last contact date</label>
            <input id="f_lastContactDate" type="date" />
          </div>
          <div class="field">
            <label>Next action due date</label>
            <input id="f_nextActionDueDate" type="date" />
          </div>
          <div class="field" style="grid-column: span 1;">
            <label>Contact email</label>
            <input id="f_contactEmail" type="email" />
          </div>
          <div class="field">
            <label>Contact phone</label>
            <input id="f_contactPhone" />
          </div>
          <div class="field" style="grid-column: span 2;">
            <label>Next action</label>
            <input id="f_nextAction" />
          </div>
          <div class="field" style="grid-column: span 3;">
            <label>Notes</label>
            <textarea id="f_notes" placeholder="Freeform notes…"></textarea>
          </div>
        </div>
        <div class="hintline">
          This file never uploads anything. Data is stored in your browser (localStorage). Use Export/Import to move data between computers.
        </div>
      </div>
      <div class="dlgFt">
        <div style="display:flex; gap:10px; align-items:center;">
          <button class="btn danger" id="btnDelete">Delete</button>
          <span class="muted" id="dlgMeta" style="font-size:11px;"></span>
        </div>
        <div style="display:flex; gap:10px;">
          <button class="btn" id="btnCancel">Cancel</button>
          <button class="btn primary" id="btnSave">Save</button>
        </div>
      </div>
    </dialog>

    <script>
      const STORAGE_KEY = "clientTracker.v1";

      const Statuses = [
        { id: "Lead", label: "Lead", badge: "lead" },
        { id: "Active", label: "Active", badge: "active" },
        { id: "Paused", label: "Paused", badge: "paused" },
        { id: "ClosedWon", label: "Closed Won", badge: "closedWon" },
        { id: "ClosedLost", label: "Closed Lost", badge: "closedLost" },
      ];

      const Stages = [
        { id: "Lead", label: "Lead" },
        { id: "DiscoveryCallBooked", label: "Discovery Call Booked" },
        { id: "ProposalSent", label: "Proposal Sent" },
        { id: "Active", label: "Active" },
        { id: "Closed", label: "Closed" },
      ];

      const $ = (id) => document.getElementById(id);
      const els = {
        q: $("q"),
        status: $("status"),
        stage: $("stage"),
        due: $("due"),
        minValue: $("minValue"),
        rows: $("rows"),
        resultMeta: $("resultMeta"),
        dash: $("dash"),
        kpiRow: $("kpiRow"),
        btnAdd: $("btnAdd"),
        btnExport: $("btnExport"),
        fileImport: $("fileImport"),
        btnReset: $("btnReset"),
        dlg: $("dlg"),
        dlgTitle: $("dlgTitle"),
        btnClose: $("btnClose"),
        btnCancel: $("btnCancel"),
        btnSave: $("btnSave"),
        btnDelete: $("btnDelete"),
        dlgMeta: $("dlgMeta"),
        f: {
          clientName: $("f_clientName"),
          company: $("f_company"),
          status: $("f_status"),
          stage: $("f_stage"),
          dealValueUsd: $("f_dealValueUsd"),
          startDate: $("f_startDate"),
          lastContactDate: $("f_lastContactDate"),
          nextAction: $("f_nextAction"),
          nextActionDueDate: $("f_nextActionDueDate"),
          contactEmail: $("f_contactEmail"),
          contactPhone: $("f_contactPhone"),
          notes: $("f_notes"),
        },
      };

      function nowIso() { return new Date().toISOString(); }
      function isoDate(d) {
        if (!d) return "";
        const x = new Date(d);
        if (Number.isNaN(x.getTime())) return "";
        return x.toISOString().slice(0, 10);
      }
      function parseDate(s) {
        if (!s) return null;
        const d = new Date(s + "T00:00:00");
        return Number.isNaN(d.getTime()) ? null : d;
      }
      function daysBetween(a, b) {
        const ms = 24 * 60 * 60 * 1000;
        return Math.floor((b - a) / ms);
      }
      function fmtMoney(n) {
        const x = Number(n || 0);
        return x.toLocaleString(undefined, { style: "currency", currency: "USD", maximumFractionDigits: 2 });
      }
      function slugify(s) {
        return String(s || "")
          .toLowerCase()
          .replace(/[^a-z0-9]+/g, "-")
          .replace(/(^-|-$)/g, "")
          .slice(0, 40) || "client";
      }
      function uid() {
        const a = crypto.getRandomValues(new Uint8Array(10));
        return Array.from(a).map(b => b.toString(16).padStart(2, "0")).join("");
      }

      function makeDefaultClient() {
        const today = isoDate(new Date());
        return {
          clientId: "c_" + uid(),
          clientName: "",
          company: "",
          contactEmail: "",
          contactPhone: "",
          status: "Lead",
          stage: "Lead",
          dealValueUsd: 0,
          startDate: today,
          lastContactDate: "",
          nextAction: "",
          nextActionDueDate: "",
          notes: "",
          createdAt: nowIso(),
          updatedAt: nowIso(),
        };
      }

      function loadState() {
        try {
          const raw = localStorage.getItem(STORAGE_KEY);
          if (!raw) return { clients: [], version: 1 };
          const data = JSON.parse(raw);
          if (!data || !Array.isArray(data.clients)) return { clients: [], version: 1 };
          return { version: 1, clients: data.clients };
        } catch {
          return { clients: [], version: 1 };
        }
      }
      function saveState() {
        localStorage.setItem(STORAGE_KEY, JSON.stringify(state));
      }

      function normalizeClient(c) {
        const d = makeDefaultClient();
        const out = { ...d, ...(c || {}) };
        out.clientId = String(out.clientId || d.clientId);
        out.clientName = String(out.clientName || "");
        out.company = String(out.company || "");
        out.contactEmail = String(out.contactEmail || "");
        out.contactPhone = String(out.contactPhone || "");
        out.status = Statuses.some(s => s.id === out.status) ? out.status : "Lead";
        out.stage = Stages.some(s => s.id === out.stage) ? out.stage : "Lead";
        out.dealValueUsd = Number(out.dealValueUsd || 0);
        out.startDate = out.startDate ? isoDate(out.startDate) : "";
        out.lastContactDate = out.lastContactDate ? isoDate(out.lastContactDate) : "";
        out.nextActionDueDate = out.nextActionDueDate ? isoDate(out.nextActionDueDate) : "";
        out.nextAction = String(out.nextAction || "");
        out.notes = String(out.notes || "");
        out.createdAt = out.createdAt || d.createdAt;
        out.updatedAt = out.updatedAt || d.updatedAt;
        return out;
      }

      let state = loadState();
      state.clients = state.clients.map(normalizeClient);
      saveState();

      let ui = {
        sortKey: "updatedAt",
        sortDir: "desc",
        editingId: null,
      };

      function setSelectOptions(sel, options, includeAllLabel) {
        sel.innerHTML = "";
        if (includeAllLabel) {
          const opt = document.createElement("option");
          opt.value = "all";
          opt.textContent = includeAllLabel;
          sel.appendChild(opt);
        }
        for (const o of options) {
          const opt = document.createElement("option");
          opt.value = o.id;
          opt.textContent = o.label;
          sel.appendChild(opt);
        }
      }

      setSelectOptions(els.status, Statuses, "All statuses");
      setSelectOptions(els.stage, Stages, "All stages");
      setSelectOptions(els.f.status, Statuses);
      setSelectOptions(els.f.stage, Stages);

      function statusBadge(status) {
        const s = Statuses.find(x => x.id === status) || Statuses[0];
        return `<span class="badge ${s.badge}">${s.label}</span>`;
      }
      function stagePill(stage) {
        const s = Stages.find(x => x.id === stage);
        return `<span class="pill">${s ? s.label : stage}</span>`;
      }

      function matchesFilters(c) {
        const q = els.q.value.trim().toLowerCase();
        if (q) {
          const hay = [c.clientName, c.company, c.contactEmail, c.contactPhone, c.nextAction, c.notes]
            .join(" ")
            .toLowerCase();
          if (!hay.includes(q)) return false;
        }
        if (els.status.value !== "all" && c.status !== els.status.value) return false;
        if (els.stage.value !== "all" && c.stage !== els.stage.value) return false;

        const minValue = Number(els.minValue.value || 0);
        if (minValue > 0 && Number(c.dealValueUsd || 0) < minValue) return false;

        const dueMode = els.due.value;
        const dueDate = parseDate(c.nextActionDueDate);
        const today = new Date();
        today.setHours(0,0,0,0);
        if (dueMode === "overdue") {
          if (!dueDate) return false;
          if (dueDate >= today) return false;
        } else if (dueMode === "none") {
          if (dueDate) return false;
        } else if (dueMode === "7" || dueMode === "30") {
          const days = Number(dueMode);
          if (!dueDate) return false;
          const delta = daysBetween(today, dueDate);
          if (delta < 0 || delta > days) return false;
        }
        return true;
      }

      function compare(a, b, key) {
        const va = a[key];
        const vb = b[key];
        if (key.includes("Date")) {
          const da = parseDate(va);
          const db = parseDate(vb);
          if (!da && !db) return 0;
          if (!da) return 1;
          if (!db) return -1;
          return da - db;
        }
        if (typeof va === "number" || typeof vb === "number") return Number(va || 0) - Number(vb || 0);
        return String(va || "").localeCompare(String(vb || ""), undefined, { sensitivity: "base" });
      }

      function getView() {
        const filtered = state.clients.filter(matchesFilters);
        filtered.sort((a, b) => {
          const r = compare(a, b, ui.sortKey);
          return ui.sortDir === "asc" ? r : -r;
        });
        return filtered;
      }

      function renderDashboard() {
        const all = state.clients;
        const total = all.length;
        const totalValue = all.reduce((s, c) => s + Number(c.dealValueUsd || 0), 0);

        const today = new Date(); today.setHours(0,0,0,0);
        const overdue = all.filter(c => {
          const d = parseDate(c.nextActionDueDate);
          return d && d < today && (c.status === "Lead" || c.status === "Active" || c.status === "Paused");
        }).length;
        const next7 = all.filter(c => {
          const d = parseDate(c.nextActionDueDate);
          if (!d) return false;
          const delta = daysBetween(today, d);
          return delta >= 0 && delta <= 7 && (c.status === "Lead" || c.status === "Active" || c.status === "Paused");
        }).length;

        const byStatus = Object.fromEntries(Statuses.map(s => [s.id, 0]));
        for (const c of all) byStatus[c.status] = (byStatus[c.status] || 0) + 1;

        els.kpiRow.innerHTML = [
          `<span class="pill"><strong>${total}</strong> clients</span>`,
          `<span class="pill"><strong>${fmtMoney(totalValue)}</strong> total value</span>`,
          `<span class="pill"><strong>${overdue}</strong> overdue actions</span>`,
          `<span class="pill"><strong>${next7}</strong> actions next 7d</span>`,
        ].join("");

        els.dash.innerHTML = [
          statCard("Lead", byStatus.Lead, "Early pipeline"),
          statCard("Active", byStatus.Active, "Currently working"),
          statCard("Paused", byStatus.Paused, "Waiting / blocked"),
          statCard("Closed Won", byStatus.ClosedWon, "Successful deals"),
          statCard("Closed Lost", byStatus.ClosedLost, "Not moving forward"),
        ].join("");
      }

      function statCard(title, value, hint) {
        return `<div class="stat"><div class="k">${title}</div><div class="v">${value}</div><div class="hint">${hint}</div></div>`;
      }

      function renderTable() {
        const view = getView();
        const total = state.clients.length;
        els.resultMeta.textContent = `${view.length} shown • ${total} total`;

        els.rows.innerHTML = view.map(c => {
          const contact = [
            c.contactEmail ? `<div class="nowrap">${escapeHtml(c.contactEmail)}</div>` : "",
            c.contactPhone ? `<div class="muted nowrap">${escapeHtml(c.contactPhone)}</div>` : "",
          ].join("");

          const last = c.lastContactDate ? `<span class="mono nowrap">${escapeHtml(c.lastContactDate)}</span>` : `<span class="muted">—</span>`;
          const nextDue = renderDue(c.nextActionDueDate, c.status);
          const nextAction = c.nextAction ? escapeHtml(c.nextAction) : `<span class="muted">—</span>`;

          return `<tr>
            <td>
              <div style="font-weight:750; letter-spacing:0.2px;">${escapeHtml(c.clientName || "Unnamed")}</div>
              <div class="muted mono" style="font-size:11px; margin-top:4px;">${escapeHtml(c.clientId)}</div>
            </td>
            <td>${escapeHtml(c.company || "—")}</td>
            <td>${statusBadge(c.status)}</td>
            <td>${stagePill(c.stage)}</td>
            <td class="nowrap">${fmtMoney(c.dealValueUsd || 0)}</td>
            <td>${last}</td>
            <td>${nextDue}</td>
            <td style="max-width: 260px;">${nextAction}</td>
            <td style="min-width: 160px;">${contact || `<span class="muted">—</span>`}</td>
            <td class="nowrap"><button class="btn small" data-edit="${escapeAttr(c.clientId)}">Edit</button></td>
          </tr>`;
        }).join("");
      }

      function renderDue(dueStr, status) {
        const d = parseDate(dueStr);
        if (!d) return `<span class="muted">—</span>`;
        const today = new Date(); today.setHours(0,0,0,0);
        const delta = daysBetween(today, d);
        const label = escapeHtml(dueStr);
        const open = (status === "Lead" || status === "Active" || status === "Paused");
        if (!open) return `<span class="mono nowrap">${label}</span>`;
        if (delta < 0) return `<span class="mono nowrap" style="color: rgba(239,68,68,0.95); font-weight:800;">${label}</span>`;
        if (delta <= 7) return `<span class="mono nowrap" style="color: rgba(245,158,11,0.95); font-weight:800;">${label}</span>`;
        return `<span class="mono nowrap">${label}</span>`;
      }

      function escapeHtml(s) {
        return String(s || "")
          .replaceAll("&", "&amp;")
          .replaceAll("<", "&lt;")
          .replaceAll(">", "&gt;")
          .replaceAll('"', "&quot;")
          .replaceAll("'", "&#39;");
      }
      function escapeAttr(s) { return escapeHtml(s).replaceAll(" ", ""); }

      function openDialog(client) {
        ui.editingId = client.clientId;
        els.dlgTitle.textContent = client.clientName ? `Edit: ${client.clientName}` : "Add client";
        els.dlgMeta.textContent = `ID: ${client.clientId} • Created: ${client.createdAt.slice(0,19).replace("T"," ")} • Updated: ${client.updatedAt.slice(0,19).replace("T"," ")}`;

        els.f.clientName.value = client.clientName || "";
        els.f.company.value = client.company || "";
        els.f.status.value = client.status || "Lead";
        els.f.stage.value = client.stage || "Lead";
        els.f.dealValueUsd.value = String(client.dealValueUsd ?? 0);
        els.f.startDate.value = client.startDate || "";
        els.f.lastContactDate.value = client.lastContactDate || "";
        els.f.nextAction.value = client.nextAction || "";
        els.f.nextActionDueDate.value = client.nextActionDueDate || "";
        els.f.contactEmail.value = client.contactEmail || "";
        els.f.contactPhone.value = client.contactPhone || "";
        els.f.notes.value = client.notes || "";

        els.btnDelete.style.visibility = state.clients.some(c => c.clientId === client.clientId) ? "visible" : "hidden";
        els.dlg.showModal();
        els.f.clientName.focus();
      }

      function closeDialog() {
        els.dlg.close();
        ui.editingId = null;
      }

      function upsertClient(patch) {
        const idx = state.clients.findIndex(c => c.clientId === patch.clientId);
        if (idx >= 0) state.clients[idx] = normalizeClient(patch);
        else state.clients.unshift(normalizeClient(patch));
        saveState();
      }

      function deleteClient(id) {
        state.clients = state.clients.filter(c => c.clientId !== id);
        saveState();
      }

      function getEditing() {
        return state.clients.find(c => c.clientId === ui.editingId) || null;
      }

      function handleSave() {
        const existing = getEditing();
        const base = existing || makeDefaultClient();
        const next = {
          ...base,
          clientName: els.f.clientName.value.trim(),
          company: els.f.company.value.trim(),
          status: els.f.status.value,
          stage: els.f.stage.value,
          dealValueUsd: Number(els.f.dealValueUsd.value || 0),
          startDate: els.f.startDate.value,
          lastContactDate: els.f.lastContactDate.value,
          nextAction: els.f.nextAction.value.trim(),
          nextActionDueDate: els.f.nextActionDueDate.value,
          contactEmail: els.f.contactEmail.value.trim(),
          contactPhone: els.f.contactPhone.value.trim(),
          notes: els.f.notes.value,
          updatedAt: nowIso(),
        };
        if (!next.clientName && !next.company) {
          alert("Please enter at least a client name or company.");
          return;
        }
        upsertClient(next);
        closeDialog();
        render();
      }

      function render() {
        renderDashboard();
        renderTable();
        renderSortIndicators();
      }

      function renderSortIndicators() {
        document.querySelectorAll("thead th[data-sort]").forEach(th => {
          th.classList.toggle("sort", th.dataset.sort === ui.sortKey);
          const key = th.dataset.sort;
          const label = th.textContent.replace(/ ↑| ↓/g, "");
          if (key === ui.sortKey) th.textContent = label + (ui.sortDir === "asc" ? " ↑" : " ↓");
          else th.textContent = label;
        });
      }

      function exportJson() {
        const payload = JSON.stringify(state, null, 2);
        const blob = new Blob([payload], { type: "application/json" });
        const url = URL.createObjectURL(blob);
        const a = document.createElement("a");
        a.href = url;
        a.download = `client-tracker-${new Date().toISOString().slice(0,10)}.json`;
        a.click();
        URL.revokeObjectURL(url);
      }

      async function importJson(file) {
        const text = await file.text();
        let data;
        try { data = JSON.parse(text); } catch { alert("Invalid JSON file."); return; }
        if (!data || !Array.isArray(data.clients)) { alert("JSON must have a top-level { clients: [...] }."); return; }
        const incoming = data.clients.map(normalizeClient);

        const mode = confirm("Import mode:\\nOK = Merge (recommended)\\nCancel = Replace all local data");
        if (mode) {
          const byId = new Map(state.clients.map(c => [c.clientId, c]));
          for (const c of incoming) byId.set(c.clientId, c);
          state.clients = Array.from(byId.values());
        } else {
          state.clients = incoming;
        }
        saveState();
        render();
      }

      function resetData() {
        const ok = confirm("This will delete local data stored in your browser for this tracker. Continue?");
        if (!ok) return;
        state = { version: 1, clients: [] };
        saveState();
        render();
      }

      function onSortClick(key) {
        if (ui.sortKey === key) ui.sortDir = ui.sortDir === "asc" ? "desc" : "asc";
        else { ui.sortKey = key; ui.sortDir = "asc"; }
        render();
      }

      // Wire up
      ["q","status","stage","due","minValue"].forEach(id => els[id].addEventListener("input", render));
      els.btnAdd.addEventListener("click", () => openDialog(makeDefaultClient()));
      els.btnExport.addEventListener("click", exportJson);
      els.fileImport.addEventListener("change", async (e) => {
        const f = e.target.files && e.target.files[0];
        e.target.value = "";
        if (f) await importJson(f);
      });
      els.btnReset.addEventListener("click", resetData);

      els.btnClose.addEventListener("click", closeDialog);
      els.btnCancel.addEventListener("click", closeDialog);
      els.btnSave.addEventListener("click", handleSave);
      els.btnDelete.addEventListener("click", () => {
        const c = getEditing();
        if (!c) return;
        const ok = confirm(`Delete ${c.clientName || c.company || "this client"}?`);
        if (!ok) return;
        deleteClient(c.clientId);
        closeDialog();
        render();
      });

      els.rows.addEventListener("click", (e) => {
        const btn = e.target.closest("[data-edit]");
        if (!btn) return;
        const id = btn.dataset.edit;
        const c = state.clients.find(x => x.clientId === id);
        if (c) openDialog(c);
      });

      document.querySelectorAll("thead th[data-sort]").forEach(th => {
        th.addEventListener("click", () => onSortClick(th.dataset.sort));
      });

      // Initial render
      render();
    </script>
  </body>
</html>
