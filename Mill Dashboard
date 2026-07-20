import { useState, useEffect, useMemo } from "react";
import * as XLSX from "xlsx";
import {
  BarChart, Bar, LineChart, Line, PieChart, Pie, Cell,
  XAxis, YAxis, Tooltip, ResponsiveContainer, Legend,
} from "recharts";

/* ============================================================
   🌾 FLOUR MILL ERP — LIVE DASHBOARD (Phase 6B)
   SETUP (sirf 2 line badalni hai):
   1. Supabase Dashboard → Settings → API → "Project URL" yahan dalo
   2. Wahi page se "anon public" key yahan dalo
   Login: Supabase → Authentication me banaya hua email/password
   ============================================================ */
const SUPABASE_URL = "https://jzphykzwofvllmlbkmqo.supabase.co";   // ← APNA URL DALO
const SUPABASE_ANON_KEY = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Imp6cGh5a3p3b2Z2bGxtbGJrbXFvIiwicm9sZSI6ImFub24iLCJpYXQiOjE3ODQ1Mzk3NDYsImV4cCI6MjEwMDExNTc0Nn0.l2lnG_IBOyrcojiYiBBpWHhD-oN1_XFOXowYSfFsmDI";                 // ← APNI ANON KEY DALO

/* ---------- Design tokens: "Mill Ledger" ---------- */
const T = {
  ink: "#1C2B24", paper: "#F4F4EF", card: "#FFFFFF",
  green: "#1E4D38", gold: "#C99A2C", red: "#B3341F",
  ok: "#2E7D4F", warn: "#B87514", line: "#E2E1D8", dim: "#6B7A70",
};
const CAT_COLORS = ["#1E4D38", "#C99A2C", "#B3341F", "#4A6FA5", "#7A5C99"];
const mono = "ui-monospace, SFMono-Regular, Menlo, monospace";

/* ---------- Supabase REST helpers ---------- */
async function sbLogin(email, password) {
  const r = await fetch(`${SUPABASE_URL}/auth/v1/token?grant_type=password`, {
    method: "POST",
    headers: { "Content-Type": "application/json", apikey: SUPABASE_ANON_KEY },
    body: JSON.stringify({ email, password }),
  });
  const d = await r.json();
  if (!r.ok) throw new Error(d.error_description || d.msg || "Login fail hua");
  return d.access_token;
}
async function sbGet(token, path) {
  const r = await fetch(`${SUPABASE_URL}/rest/v1/${path}`, {
    headers: { apikey: SUPABASE_ANON_KEY, Authorization: `Bearer ${token}` },
  });
  if (!r.ok) throw new Error(`${path.split("?")[0]} load nahi hua (${r.status})`);
  return r.json();
}

/* ---------- Chhote reusable parts ---------- */
const S = { // shared styles
  card: { background: T.card, border: `1px solid ${T.line}`, borderRadius: 10, padding: 14 },
  h: { margin: "0 0 10px", fontSize: 13, fontWeight: 700, letterSpacing: ".08em",
       textTransform: "uppercase", color: T.dim },
  td: { padding: "7px 8px", borderBottom: `1px solid ${T.line}`, fontSize: 13 },
  th: { padding: "7px 8px", borderBottom: `2px solid ${T.ink}`, fontSize: 11, textAlign: "left",
        textTransform: "uppercase", letterSpacing: ".05em", color: T.dim },
  input: { padding: "9px 10px", border: `1px solid ${T.line}`, borderRadius: 8,
           fontSize: 14, background: "#fff", color: T.ink, width: "100%", boxSizing: "border-box" },
  btn: { padding: "10px 16px", border: "none", borderRadius: 8, background: T.green,
         color: "#fff", fontSize: 14, fontWeight: 600, cursor: "pointer" },
};

function DataTable({ cols, rows, format = {} }) {
  if (!rows?.length) return <div style={{ color: T.dim, fontSize: 13, padding: 8 }}>Koi data nahi — filter badal ke dekho.</div>;
  return (
    <div style={{ overflowX: "auto" }}>
      <table style={{ borderCollapse: "collapse", width: "100%", minWidth: cols.length * 90 }}>
        <thead><tr>{cols.map((c) => <th key={c} style={S.th}>{c.replaceAll("_", " ")}</th>)}</tr></thead>
        <tbody>
          {rows.map((r, i) => (
            <tr key={i}>
              {cols.map((c) => {
                const v = r[c];
                const isNum = typeof v === "number" || (!isNaN(parseFloat(v)) && String(v).match(/^-?[\d.]+$/));
                return (
                  <td key={c} style={{ ...S.td, fontFamily: isNum ? mono : "inherit",
                    color: String(v).includes("🔴") ? T.red : String(v).includes("🟠") || String(v).includes("🟡") ? T.warn : T.ink }}>
                    {format[c] ? format[c](v) : v ?? "—"}
                  </td>
                );
              })}
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}

const inr = (v) => "₹" + Number(v || 0).toLocaleString("en-IN");

/* ============================================================ MAIN APP */
export default function MillDashboard() {
  const [token, setToken] = useState(null);
  const [tab, setTab] = useState("aaj");
  const [err, setErr] = useState("");
  const [loading, setLoading] = useState(false);
  const [D, setD] = useState({}); // saara data yahan

  async function loadAll(tk) {
    setLoading(true); setErr("");
    const views = [
      "v_dashboard_today?order=sort_no", "v_exception_report",
      "v_daily_production_30d", "v_daily_sales_30d",
      "v_current_stock", "v_order_suggestions", "v_pm_stock",
      "v_yield_report?limit=15", "v_outstanding_aging",
      "v_maintenance_due", "v_machine_cost",
    ];
    try {
      const out = {};
      for (const v of views) out[v.split("?")[0]] = await sbGet(tk, v);
      setD(out);
    } catch (e) { setErr(e.message); }
    setLoading(false);
  }

  useEffect(() => { if (token) loadAll(token); }, [token]);

  if (!token) return <Login onLogin={(tk) => setToken(tk)} />;

  const tabs = [
    ["aaj", "📊 Aaj"], ["stock", "📦 Stock"], ["report", "📑 Reports"],
    ["paisa", "💰 Paisa"], ["machine", "🔧 Machine"],
  ];

  return (
    <div style={{ minHeight: "100vh", background: T.paper, color: T.ink,
      fontFamily: "ui-sans-serif, system-ui, sans-serif" }}>
      {/* Header */}
      <div style={{ background: T.green, color: "#fff", padding: "12px 16px",
        display: "flex", alignItems: "center", justifyContent: "space-between", flexWrap: "wrap", gap: 8 }}>
        <div>
          <div style={{ fontSize: 11, letterSpacing: ".2em", opacity: 0.8 }}>ROLLER FLOUR MILL</div>
          <div style={{ fontSize: 19, fontWeight: 800 }}>Live Dashboard</div>
        </div>
        <button onClick={() => loadAll(token)} style={{ ...S.btn, background: T.gold, color: T.ink }}>
          {loading ? "Load ho raha..." : "🔄 Refresh"}
        </button>
      </div>

      {/* Tabs */}
      <div style={{ display: "flex", gap: 6, padding: "10px 12px", overflowX: "auto",
        borderBottom: `1px solid ${T.line}`, background: "#fff" }}>
        {tabs.map(([k, label]) => (
          <button key={k} onClick={() => setTab(k)} style={{
            padding: "8px 14px", borderRadius: 20, fontSize: 13, fontWeight: 600, whiteSpace: "nowrap",
            border: `1px solid ${tab === k ? T.green : T.line}`, cursor: "pointer",
            background: tab === k ? T.green : "#fff", color: tab === k ? "#fff" : T.ink }}>
            {label}
          </button>
        ))}
      </div>

      {err && <div style={{ margin: 12, padding: 12, background: "#FBEAE7", color: T.red,
        borderRadius: 8, fontSize: 13 }}>⚠️ {err} — SUPABASE_URL/KEY check karo, aur Phase 1–6A ke SQL chale hain kya?</div>}

      <div style={{ padding: 12, display: "grid", gap: 12, maxWidth: 1100, margin: "0 auto" }}>
        {tab === "aaj" && <TabAaj D={D} />}
        {tab === "stock" && <TabStock D={D} />}
        {tab === "report" && <TabReport token={token} />}
        {tab === "paisa" && <TabPaisa D={D} />}
        {tab === "machine" && <TabMachine D={D} />}
      </div>
    </div>
  );
}

/* ---------- LOGIN ---------- */
function Login({ onLogin }) {
  const [email, setEmail] = useState(""); const [pw, setPw] = useState("");
  const [msg, setMsg] = useState(""); const [busy, setBusy] = useState(false);
  async function go() {
    setBusy(true); setMsg("");
    try { onLogin(await sbLogin(email, pw)); }
    catch (e) { setMsg(e.message); }
    setBusy(false);
  }
  return (
    <div style={{ minHeight: "100vh", background: T.green, display: "flex",
      alignItems: "center", justifyContent: "center", padding: 16,
      fontFamily: "ui-sans-serif, system-ui, sans-serif" }}>
      <div style={{ ...S.card, width: 340, padding: 24 }}>
        <div style={{ fontSize: 30, textAlign: "center" }}>🌾</div>
        <h2 style={{ textAlign: "center", margin: "6px 0 2px" }}>Mill Dashboard</h2>
        <p style={{ textAlign: "center", color: T.dim, fontSize: 13, marginTop: 0 }}>
          Apne Supabase user se login karo</p>
        <div style={{ display: "grid", gap: 10 }}>
          <input style={S.input} placeholder="Email" value={email} onChange={(e) => setEmail(e.target.value)} />
          <input style={S.input} placeholder="Password" type="password" value={pw}
            onChange={(e) => setPw(e.target.value)} onKeyDown={(e) => e.key === "Enter" && go()} />
          <button style={S.btn} onClick={go} disabled={busy}>{busy ? "Ruko..." : "Login"}</button>
          {msg && <div style={{ color: T.red, fontSize: 13 }}>{msg}</div>}
          {SUPABASE_URL.includes("YOUR-") &&
            <div style={{ fontSize: 12, color: T.warn }}>⚠️ Pehle code me SUPABASE_URL aur ANON_KEY dalo (sabse upar 2 lines).</div>}
        </div>
      </div>
    </div>
  );
}

/* ---------- TAB: AAJ ---------- */
function TabAaj({ D }) {
  const today = D.v_dashboard_today || [];
  const exc = D.v_exception_report || [];
  const prod = D.v_daily_production_30d || [];
  const sales = D.v_daily_sales_30d || [];
  const byCat = useMemo(() => {
    const m = {};
    sales.forEach((s) => { m[s.category] = (m[s.category] || 0) + Number(s.sales_amount); });
    return Object.entries(m).map(([name, value]) => ({ name, value }));
  }, [sales]);

  return (<>
    {/* 🚨 SIGNATURE: Exception Board — mandi blackboard style */}
    <div style={{ background: T.ink, color: "#F2EFE4", borderRadius: 10, padding: 14 }}>
      <div style={{ fontSize: 12, letterSpacing: ".15em", color: T.gold, fontWeight: 700 }}>
        🚨 EXCEPTION BOARD — {exc.length} DHYAN DENE WALI BAATEIN
      </div>
      {exc.length === 0 ? (
        <div style={{ marginTop: 8, fontSize: 14 }}>✅ Aaj sab theek hai — koi red flag nahi.</div>
      ) : (
        <div style={{ marginTop: 8, display: "grid", gap: 6 }}>
          {exc.map((e, i) => (
            <div key={i} style={{ display: "flex", gap: 8, fontSize: 13, lineHeight: 1.45,
              borderLeft: `3px solid ${T.red}`, paddingLeft: 8 }}>
              <span style={{ color: T.gold, fontFamily: mono, fontSize: 11, minWidth: 88 }}>{e.area}</span>
              <span><b>{e.ref}</b> — {e.problem}</span>
            </div>
          ))}
        </div>
      )}
    </div>

    {/* Aaj ke numbers */}
    <div style={{ display: "grid", gridTemplateColumns: "repeat(auto-fit,minmax(150px,1fr))", gap: 10 }}>
      {today.map((m) => (
        <div key={m.metric} style={{ ...S.card, borderTop: `3px solid ${m.unit === "₹" ? T.gold : T.green}` }}>
          <div style={{ fontSize: 11, color: T.dim, textTransform: "uppercase", letterSpacing: ".04em" }}>{m.metric}</div>
          <div style={{ fontSize: 22, fontWeight: 800, fontFamily: mono, marginTop: 4 }}>
            {m.unit === "₹" ? inr(m.value) : `${Number(m.value).toLocaleString("en-IN")} ${m.unit}`}
          </div>
        </div>
      ))}
    </div>

    {/* Charts */}
    <div style={{ display: "grid", gridTemplateColumns: "repeat(auto-fit,minmax(280px,1fr))", gap: 12 }}>
      <div style={S.card}>
        <h3 style={S.h}>Pichle 30 din — Daily Grinding (qtl)</h3>
        <ResponsiveContainer width="100%" height={200}>
          <BarChart data={prod}>
            <XAxis dataKey="prod_date" tick={{ fontSize: 10 }} tickFormatter={(d) => d?.slice(5)} />
            <YAxis tick={{ fontSize: 10 }} /><Tooltip />
            <Bar dataKey="wheat_ground_qtl" fill={T.green} radius={[3, 3, 0, 0]} />
          </BarChart>
        </ResponsiveContainer>
      </div>
      <div style={S.card}>
        <h3 style={S.h}>Sales — Corporate vs Retail vs Market (30 din)</h3>
        <ResponsiveContainer width="100%" height={200}>
          <PieChart>
            <Pie data={byCat} dataKey="value" nameKey="name" outerRadius={70} label={(e) => e.name}>
              {byCat.map((_, i) => <Cell key={i} fill={CAT_COLORS[i % CAT_COLORS.length]} />)}
            </Pie>
            <Tooltip formatter={(v) => inr(v)} /><Legend />
          </PieChart>
        </ResponsiveContainer>
      </div>
    </div>

    <div style={S.card}>
      <h3 style={S.h}>Batch-wise Yield (chori/wastage yahin dikhti hai)</h3>
      <DataTable cols={["batch_no", "prod_date", "total_input_qtl", "total_output_qtl", "yield_pct", "flag"]}
        rows={D.v_yield_report} />
    </div>
  </>);
}

/* ---------- TAB: STOCK ---------- */
function TabStock({ D }) {
  return (<>
    <div style={S.card}>
      <h3 style={S.h}>🔔 Order Suggestions (10 din lead time ke saath)</h3>
      <DataTable cols={["name", "current_stock", "daily_use", "days_left", "reorder_level", "status", "suggested_order_qty"]}
        rows={D.v_order_suggestions} />
    </div>
    <div style={{ display: "grid", gridTemplateColumns: "repeat(auto-fit,minmax(280px,1fr))", gap: 12 }}>
      <div style={S.card}>
        <h3 style={S.h}>Godown-wise Current Stock</h3>
        <DataTable cols={["item_name", "warehouse", "stock_qty"]} rows={D.v_current_stock} />
      </div>
      <div style={S.card}>
        <h3 style={S.h}>PM Stock (Brand-wise bags)</h3>
        <DataTable cols={["name", "stock_qty", "uom"]} rows={D.v_pm_stock} />
      </div>
    </div>
  </>);
}

/* ---------- TAB: DYNAMIC REPORT (SKU × Party + Excel) ---------- */
function TabReport({ token }) {
  const [from, setFrom] = useState(""); const [to, setTo] = useState("");
  const [party, setParty] = useState(""); const [sku, setSku] = useState("");
  const [cat, setCat] = useState(""); const [groupBy, setGroupBy] = useState("none");
  const [rows, setRows] = useState([]); const [busy, setBusy] = useState(false); const [msg, setMsg] = useState("");

  async function run() {
    setBusy(true); setMsg("");
    let q = "v_sales_dynamic?select=*&order=invoice_date.desc";
    if (from) q += `&invoice_date=gte.${from}`;
    if (to) q += `&invoice_date=lte.${to}`;
    if (party) q += `&party=ilike.*${party}*`;
    if (sku) q += `&sku=ilike.*${sku}*`;
    if (cat) q += `&category=eq.${cat}`;
    try { setRows(await sbGet(token, q)); } catch (e) { setMsg(e.message); }
    setBusy(false);
  }
  useEffect(() => { run(); }, []);

  const grouped = useMemo(() => {
    if (groupBy === "none") return null;
    const key = (r) => groupBy === "party" ? r.party : groupBy === "sku" ? r.sku
      : groupBy === "party_sku" ? `${r.party} | ${r.sku}` : r.category;
    const m = {};
    rows.forEach((r) => {
      const k = key(r);
      if (!m[k]) m[k] = { group: k, bags: 0, amount: 0, invoices: 0 };
      m[k].bags += Number(r.qty_bags); m[k].amount += Number(r.total_amount); m[k].invoices += 1;
    });
    return Object.values(m).sort((a, b) => b.amount - a.amount);
  }, [rows, groupBy]);

  function downloadExcel() {
    const data = grouped ?? rows;
    if (!data.length) { setMsg("Pehle report chalao, phir download karo."); return; }
    const ws = XLSX.utils.json_to_sheet(data);
    const wb = XLSX.utils.book_new();
    XLSX.utils.book_append_sheet(wb, ws, "Report");
    XLSX.writeFile(wb, `mill-report-${new Date().toISOString().slice(0, 10)}.xlsx`);
  }

  const fl = { display: "grid", gap: 4, fontSize: 12, color: T.dim };
  return (
    <div style={S.card}>
      <h3 style={S.h}>📑 Dynamic Report — SKU × Party × Date (jo chaho filter lagao)</h3>
      <div style={{ display: "grid", gridTemplateColumns: "repeat(auto-fit,minmax(140px,1fr))", gap: 10, marginBottom: 10 }}>
        <label style={fl}>Date se<input type="date" style={S.input} value={from} onChange={(e) => setFrom(e.target.value)} /></label>
        <label style={fl}>Date tak<input type="date" style={S.input} value={to} onChange={(e) => setTo(e.target.value)} /></label>
        <label style={fl}>Party<input style={S.input} placeholder="e.g. Britannia" value={party} onChange={(e) => setParty(e.target.value)} /></label>
        <label style={fl}>SKU/Item<input style={S.input} placeholder="e.g. Maida 50kg" value={sku} onChange={(e) => setSku(e.target.value)} /></label>
        <label style={fl}>Category
          <select style={S.input} value={cat} onChange={(e) => setCat(e.target.value)}>
            <option value="">Sab</option><option value="corporate">Corporate</option>
            <option value="retail">Retail</option><option value="restaurant">Restaurant</option>
            <option value="open_market">Open Market</option>
          </select></label>
        <label style={fl}>Group by
          <select style={S.input} value={groupBy} onChange={(e) => setGroupBy(e.target.value)}>
            <option value="none">Detail (invoice-wise)</option><option value="party">Party-wise</option>
            <option value="sku">SKU-wise</option><option value="party_sku">Party × SKU</option>
            <option value="category">Category-wise</option>
          </select></label>
      </div>
      <div style={{ display: "flex", gap: 8, flexWrap: "wrap", marginBottom: 12 }}>
        <button style={S.btn} onClick={run} disabled={busy}>{busy ? "Ruko..." : "▶ Report Chalao"}</button>
        <button style={{ ...S.btn, background: T.gold, color: T.ink }} onClick={downloadExcel}>📥 Excel Download</button>
      </div>
      {msg && <div style={{ color: T.red, fontSize: 13, marginBottom: 8 }}>{msg}</div>}
      {grouped
        ? <DataTable cols={["group", "bags", "invoices", "amount"]} rows={grouped} format={{ amount: inr }} />
        : <DataTable cols={["invoice_date", "party", "category", "sku", "batch_no", "qty_bags", "rate", "total_amount", "vehicle_no"]}
            rows={rows} format={{ total_amount: inr, rate: inr }} />}
    </div>
  );
}

/* ---------- TAB: PAISA ---------- */
function TabPaisa({ D }) {
  const aging = D.v_outstanding_aging || [];
  const total = aging.reduce((s, r) => s + Number(r.balance), 0);
  return (<>
    <div style={{ ...S.card, borderTop: `3px solid ${T.gold}` }}>
      <div style={{ fontSize: 11, color: T.dim, textTransform: "uppercase" }}>Total Bakaya (Outstanding)</div>
      <div style={{ fontSize: 28, fontWeight: 800, fontFamily: mono }}>{inr(total)}</div>
    </div>
    <div style={S.card}>
      <h3 style={S.h}>Party-wise Outstanding + Aging</h3>
      <DataTable cols={["party", "invoice_no", "invoice_date", "due_date", "total_amount", "paid", "balance", "aging_bucket"]}
        rows={aging} format={{ total_amount: inr, paid: inr, balance: inr }} />
    </div>
  </>);
}

/* ---------- TAB: MACHINE ---------- */
function TabMachine({ D }) {
  return (<>
    <div style={S.card}>
      <h3 style={S.h}>⏰ Maintenance Due (kharab hone se pehle karo)</h3>
      <DataTable cols={["machine", "task", "last_done", "next_due", "days_left", "status"]} rows={D.v_maintenance_due} />
    </div>
    <div style={S.card}>
      <h3 style={S.h}>🔧 Machine-wise Kharcha + Breakdown</h3>
      <DataTable cols={["machine", "spare_cost", "labour_cost", "total_cost", "breakdown_count", "downtime_hours", "flag"]}
        rows={D.v_machine_cost} format={{ spare_cost: inr, labour_cost: inr, total_cost: inr }} />
    </div>
  </>);
}
