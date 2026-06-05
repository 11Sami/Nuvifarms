import { useState } from "react";

const GOOGLE_SCRIPT_URL = ""; // Manager will paste their deployed Apps Script URL here

const MANAGER_PASSWORD = "nuvi2026";

const crops = ["Groundnut", "Soyabeans", "Beans", "Maize", "Guinea Corn", "Rice"];
const repaymentMap = {
  Groundnut: "4 bags",
  Soyabeans: "2 bags",
  Beans: "2 bags",
  Maize: "3 bags",
  "Guinea Corn": "3 bags",
  Rice: "3 bags",
};

const DB_KEY = "nuvi_applications";

function loadApps() {
  try {
    const raw = localStorage.getItem(DB_KEY);
    return raw ? JSON.parse(raw) : [];
  } catch { return []; }
}

function saveApps(apps) {
  localStorage.setItem(DB_KEY, JSON.stringify(apps));
}

// ─── Leaf SVG decoration ───────────────────────────────────────────────────
function LeafDeco({ style }) {
  return (
    <svg viewBox="0 0 80 80" style={{ opacity: 0.13, ...style }} fill="none">
      <path d="M10 70 Q20 10 70 10 Q60 40 40 50 Q25 58 10 70Z" fill="#4caf50"/>
    </svg>
  );
}

// ─── LOGO ─────────────────────────────────────────────────────────────────
function Logo({ size = 54 }) {
  return (
    <svg width={size} height={size} viewBox="0 0 100 100">
      <polygon points="50,5 95,90 5,90" fill="none" stroke="#2e7d32" strokeWidth="6"/>
      <polygon points="50,18 83,83 17,83" fill="#2e7d32" opacity="0.15"/>
      <text x="50" y="60" textAnchor="middle" fontSize="11" fontWeight="bold" fill="#2e7d32" fontFamily="serif">NUVI</text>
      <text x="50" y="74" textAnchor="middle" fontSize="7" fill="#388e3c" fontFamily="serif">FARMERS</text>
    </svg>
  );
}

// ─── STATUS BADGE ──────────────────────────────────────────────────────────
function Badge({ status }) {
  const map = {
    Pending:  { bg: "#fff8e1", color: "#f57f17", border: "#ffe082" },
    Approved: { bg: "#e8f5e9", color: "#2e7d32", border: "#a5d6a7" },
    Rejected: { bg: "#fce4ec", color: "#b71c1c", border: "#ef9a9a" },
  };
  const s = map[status] || map.Pending;
  return (
    <span style={{
      background: s.bg, color: s.color, border: `1px solid ${s.border}`,
      borderRadius: 20, padding: "3px 14px", fontSize: 12, fontWeight: 700,
      letterSpacing: 1, textTransform: "uppercase"
    }}>{status}</span>
  );
}

// ─── INPUT ─────────────────────────────────────────────────────────────────
function Field({ label, required, children, hint }) {
  return (
    <div style={{ marginBottom: 20 }}>
      <label style={{ display: "block", fontWeight: 700, marginBottom: 6,
        fontSize: 13, color: "#1b5e20", letterSpacing: 0.5, textTransform: "uppercase" }}>
        {label}{required && <span style={{ color: "#c62828", marginLeft: 3 }}>*</span>}
      </label>
      {children}
      {hint && <p style={{ margin: "4px 0 0", fontSize: 11, color: "#666" }}>{hint}</p>}
    </div>
  );
}

const inputStyle = {
  width: "100%", boxSizing: "border-box", padding: "11px 14px",
  border: "1.5px solid #c8e6c9", borderRadius: 8, fontSize: 14,
  background: "#f9fbe7", color: "#1a1a1a", outline: "none",
  fontFamily: "inherit", transition: "border 0.2s"
};

// ═══════════════════════════════════════════════════════════════════════════
//  CUSTOMER FORM
// ═══════════════════════════════════════════════════════════════════════════
function CustomerForm({ onDone }) {
  const [step, setStep] = useState(1);
  const [agreed, setAgreed] = useState(false);
  const [submitting, setSubmitting] = useState(false);
  const [form, setForm] = useState({
    fullName: "", phone: "", whatsapp: "", email: "",
    dob: "", address: "", nin: "", bvn: "",
    cropType: "", empowermentType: "Chemical (1 Carton)",
    guarantorName: "", guarantorPhone: "", guarantorAddress: "",
    guarantorNin: "",
  });

  const set = (k) => (e) => setForm(f => ({ ...f, [k]: e.target.value }));

  function handleSubmit() {
    setSubmitting(true);
    const apps = loadApps();
    const newApp = {
      id: "NUVI-" + Date.now(),
      ...form,
      repayment: repaymentMap[form.cropType] || "N/A",
      status: "Pending",
      submittedAt: new Date().toISOString(),
      managerNote: "",
    };
    apps.push(newApp);
    saveApps(apps);
    setTimeout(() => { setSubmitting(false); onDone(newApp); }, 900);
  }

  const repay = repaymentMap[form.cropType];

  return (
    <div style={{ minHeight: "100vh", background: "linear-gradient(135deg,#e8f5e9 0%,#f9fbe7 60%,#fff8e1 100%)", padding: "0 0 60px" }}>
      {/* Header */}
      <div style={{ background: "#1b5e20", padding: "18px 24px", display: "flex", alignItems: "center", gap: 16 }}>
        <Logo size={48}/>
        <div>
          <div style={{ color: "#fff", fontFamily: "Georgia,serif", fontWeight: 900, fontSize: 17, lineHeight: 1.2 }}>
            NUVI FARMERS PROGRESS COMPANY
          </div>
          <div style={{ color: "#a5d6a7", fontSize: 11, letterSpacing: 1 }}>INTERNATIONAL LTD. — Empowerment Application</div>
        </div>
      </div>

      {/* Progress bar */}
      <div style={{ background: "#2e7d32", height: 4, width: "100%" }}>
        <div style={{ background: "#76ff03", height: 4, width: `${(step/3)*100}%`, transition: "width 0.4s" }}/>
      </div>
      <div style={{ textAlign: "center", padding: "10px", fontSize: 12, color: "#2e7d32", fontWeight: 700 }}>
        STEP {step} OF 3 — {["", "Personal Information", "Crop & Empowerment", "Guarantor & Agreement"][step]}
      </div>

      <div style={{ maxWidth: 620, margin: "0 auto", padding: "0 16px" }}>

        {/* ── STEP 1 ── */}
        {step === 1 && (
          <div style={{ background: "#fff", borderRadius: 16, padding: "28px 28px", boxShadow: "0 4px 24px rgba(0,0,0,0.08)" }}>
            <h2 style={{ margin: "0 0 20px", color: "#1b5e20", fontFamily: "Georgia,serif", fontSize: 20 }}>👤 Personal Information</h2>

            <Field label="Full Name" required>
              <input style={inputStyle} value={form.fullName} onChange={set("fullName")} placeholder="e.g. John Adamu Bello"/>
            </Field>
            <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 14 }}>
              <Field label="Phone Number" required>
                <input style={inputStyle} value={form.phone} onChange={set("phone")} placeholder="+234 800 000 0000"/>
              </Field>
              <Field label="WhatsApp Number" required>
                <input style={inputStyle} value={form.whatsapp} onChange={set("whatsapp")} placeholder="+234 800 000 0000"/>
              </Field>
            </div>
            <Field label="Email Address" hint="Optional — for digital notifications">
              <input style={inputStyle} value={form.email} onChange={set("email")} placeholder="you@email.com"/>
            </Field>
            <Field label="Date of Birth" required hint="Must be 18 years or older">
              <input style={{ ...inputStyle }} type="date" value={form.dob} onChange={set("dob")}/>
            </Field>
            <Field label="Permanent Home Address" required>
              <textarea style={{ ...inputStyle, minHeight: 70, resize: "vertical" }} value={form.address} onChange={set("address")} placeholder="Full address including LGA and State"/>
            </Field>
            <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 14 }}>
              <Field label="NIN" required hint="11-digit National ID Number">
                <input style={inputStyle} value={form.nin} onChange={set("nin")} placeholder="12345678901" maxLength={11}/>
              </Field>
              <Field label="BVN" required hint="11-digit Bank Verification Number">
                <input style={inputStyle} value={form.bvn} onChange={set("bvn")} placeholder="12345678901" maxLength={11}/>
              </Field>
            </div>
            <button onClick={() => setStep(2)} disabled={!form.fullName || !form.phone || !form.address || !form.nin || !form.bvn || !form.dob}
              style={{ width: "100%", padding: "14px", background: !form.fullName || !form.phone || !form.address || !form.nin || !form.bvn || !form.dob ? "#c8e6c9" : "#2e7d32",
                color: "#fff", border: "none", borderRadius: 10, fontWeight: 800, fontSize: 15, cursor: "pointer", letterSpacing: 1 }}>
              CONTINUE →
            </button>
          </div>
        )}

        {/* ── STEP 2 ── */}
        {step === 2 && (
          <div style={{ background: "#fff", borderRadius: 16, padding: "28px 28px", boxShadow: "0 4px 24px rgba(0,0,0,0.08)" }}>
            <h2 style={{ margin: "0 0 20px", color: "#1b5e20", fontFamily: "Georgia,serif", fontSize: 20 }}>🌾 Crop & Empowerment Details</h2>

            <Field label="Type of Crop" required>
              <select style={inputStyle} value={form.cropType} onChange={set("cropType")}>
                <option value="">— Select your crop —</option>
                {crops.map(c => <option key={c} value={c}>{c}</option>)}
              </select>
            </Field>

            {repay && (
              <div style={{ background: "#e8f5e9", border: "1.5px solid #a5d6a7", borderRadius: 10, padding: "14px 18px", marginBottom: 20 }}>
                <div style={{ fontWeight: 700, color: "#1b5e20", marginBottom: 4 }}>📋 Repayment Obligation</div>
                <div style={{ fontSize: 14, color: "#333" }}>
                  For <strong>{form.cropType}</strong>, you will repay <strong>{repay}</strong> of harvested crop on or before <strong>20th December</strong>.
                </div>
              </div>
            )}

            <Field label="Empowerment Type" required>
              <select style={inputStyle} value={form.empowermentType} onChange={set("empowermentType")}>
                <option value="Chemical (1 Carton)">Chemical — 1 Carton</option>
                <option value="Cash Equivalent">Cash Equivalent</option>
              </select>
            </Field>

            <div style={{ background: "#fff8e1", border: "1px solid #ffe082", borderRadius: 10, padding: "14px 18px", marginBottom: 24, fontSize: 13, color: "#5d4037" }}>
              <strong>⏰ Important Dates:</strong><br/>
              Distribution: <strong>15th May – 15th June</strong><br/>
              Repayment deadline: <strong>20th December</strong>
            </div>

            <div style={{ display: "flex", gap: 12 }}>
              <button onClick={() => setStep(1)} style={{ flex: 1, padding: "13px", background: "#f1f8e9", color: "#2e7d32", border: "1.5px solid #a5d6a7", borderRadius: 10, fontWeight: 700, cursor: "pointer" }}>← BACK</button>
              <button onClick={() => setStep(3)} disabled={!form.cropType}
                style={{ flex: 2, padding: "13px", background: !form.cropType ? "#c8e6c9" : "#2e7d32", color: "#fff", border: "none", borderRadius: 10, fontWeight: 800, fontSize: 15, cursor: "pointer" }}>
                CONTINUE →
              </button>
            </div>
          </div>
        )}

        {/* ── STEP 3 ── */}
        {step === 3 && (
          <div style={{ background: "#fff", borderRadius: 16, padding: "28px 28px", boxShadow: "0 4px 24px rgba(0,0,0,0.08)" }}>
            <h2 style={{ margin: "0 0 20px", color: "#1b5e20", fontFamily: "Georgia,serif", fontSize: 20 }}>🤝 Guarantor & Agreement</h2>

            <Field label="Guarantor Full Name" required>
              <input style={inputStyle} value={form.guarantorName} onChange={set("guarantorName")} placeholder="Guarantor's legal name"/>
            </Field>
            <Field label="Guarantor Phone Number" required>
              <input style={inputStyle} value={form.guarantorPhone} onChange={set("guarantorPhone")} placeholder="+234 800 000 0000"/>
            </Field>
            <Field label="Guarantor Home Address" required>
              <textarea style={{ ...inputStyle, minHeight: 60, resize: "vertical" }} value={form.guarantorAddress} onChange={set("guarantorAddress")} placeholder="Guarantor's permanent address"/>
            </Field>
            <Field label="Guarantor NIN" required>
              <input style={inputStyle} value={form.guarantorNin} onChange={set("guarantorNin")} placeholder="11-digit NIN" maxLength={11}/>
            </Field>

            {/* Terms box */}
            <div style={{ background: "#f9fbe7", border: "1.5px solid #c8e6c9", borderRadius: 10, padding: "16px", marginBottom: 18, maxHeight: 200, overflowY: "auto", fontSize: 13, color: "#333", lineHeight: 1.7 }}>
              <strong style={{ color: "#1b5e20" }}>TERMS & CONDITIONS — NUVI FARMERS PROGRESS COMPANY INTERNATIONAL LTD.</strong><br/><br/>
              1. Empowerment is by giving either 1 carton of chemical or cash equivalent. Distribution is from 15th May to 15th June. Repayment is in bags of harvested crop on or before 20th December as follows: Groundnut — 4 bags, Soyabeans — 2 bags, Beans — 2 bags, Maize — 3 bags, Guinea Corn — 3 bags, Rice — 3 bags.<br/><br/>
              2. The customer must submit NIN and BVN printouts and provide reliable, accurate and complete information: name, valid phone number, permanent home address, passport photograph, and a guarantor with the same requirements.<br/><br/>
              3. Customer must be 18 years or older.<br/><br/>
              4. Customer must comply with all conditions in this agreement.<br/><br/>
              5. Legal action shall be taken for any breach of these terms and conditions.
            </div>

            <label style={{ display: "flex", alignItems: "flex-start", gap: 10, cursor: "pointer", marginBottom: 22 }}>
              <input type="checkbox" checked={agreed} onChange={e => setAgreed(e.target.checked)} style={{ marginTop: 3, width: 18, height: 18 }}/>
              <span style={{ fontSize: 13, color: "#333", lineHeight: 1.6 }}>
                I have read and agree to the Terms & Conditions. I understand that legal action may be taken against me for breach of this agreement.
              </span>
            </label>

            <div style={{ display: "flex", gap: 12 }}>
              <button onClick={() => setStep(2)} style={{ flex: 1, padding: "13px", background: "#f1f8e9", color: "#2e7d32", border: "1.5px solid #a5d6a7", borderRadius: 10, fontWeight: 700, cursor: "pointer" }}>← BACK</button>
              <button onClick={handleSubmit}
                disabled={!agreed || !form.guarantorName || !form.guarantorPhone || !form.guarantorAddress || !form.guarantorNin || submitting}
                style={{ flex: 2, padding: "13px",
                  background: (!agreed || !form.guarantorName || !form.guarantorPhone || !form.guarantorAddress || !form.guarantorNin) ? "#c8e6c9" : "#1b5e20",
                  color: "#fff", border: "none", borderRadius: 10, fontWeight: 800, fontSize: 15, cursor: "pointer" }}>
                {submitting ? "Submitting..." : "✅ SUBMIT APPLICATION"}
              </button>
            </div>
          </div>
        )}
      </div>
    </div>
  );
}

// ═══════════════════════════════════════════════════════════════════════════
//  SUCCESS SCREEN
// ═══════════════════════════════════════════════════════════════════════════
function SuccessScreen({ app, onReset }) {
  return (
    <div style={{ minHeight: "100vh", background: "linear-gradient(135deg,#e8f5e9,#f9fbe7)", display: "flex", alignItems: "center", justifyContent: "center", padding: 20 }}>
      <div style={{ background: "#fff", borderRadius: 20, padding: "40px 32px", maxWidth: 480, width: "100%", textAlign: "center", boxShadow: "0 8px 40px rgba(0,0,0,0.1)" }}>
        <div style={{ fontSize: 60, marginBottom: 10 }}>🌱</div>
        <h2 style={{ color: "#1b5e20", fontFamily: "Georgia,serif", margin: "0 0 10px" }}>Application Submitted!</h2>
        <p style={{ color: "#555", fontSize: 14, marginBottom: 20 }}>
          Your application has been received and is pending review by NUVI Management.
        </p>
        <div style={{ background: "#f1f8e9", borderRadius: 10, padding: "16px", marginBottom: 24, textAlign: "left" }}>
          <div style={{ fontSize: 12, color: "#888", marginBottom: 4 }}>APPLICATION REFERENCE</div>
          <div style={{ fontWeight: 800, color: "#1b5e20", fontSize: 16, fontFamily: "monospace" }}>{app.id}</div>
          <div style={{ marginTop: 10, fontSize: 13, color: "#555" }}>
            <strong>Name:</strong> {app.fullName}<br/>
            <strong>Crop:</strong> {app.cropType}<br/>
            <strong>Empowerment:</strong> {app.empowermentType}<br/>
            <strong>Repayment:</strong> {app.repayment}
          </div>
        </div>
        <p style={{ fontSize: 12, color: "#888", marginBottom: 20 }}>
          Please keep your reference number safe. You will be contacted via your phone or WhatsApp when your application is reviewed.
        </p>
        <button onClick={onReset} style={{ padding: "12px 32px", background: "#2e7d32", color: "#fff", border: "none", borderRadius: 10, fontWeight: 700, cursor: "pointer" }}>
          Submit Another Application
        </button>
      </div>
    </div>
  );
}

// ═══════════════════════════════════════════════════════════════════════════
//  MANAGER DASHBOARD
// ═══════════════════════════════════════════════════════════════════════════
function ManagerDashboard({ onExit }) {
  const [apps, setApps] = useState(loadApps());
  const [selected, setSelected] = useState(null);
  const [note, setNote] = useState("");
  const [filter, setFilter] = useState("All");
  const [search, setSearch] = useState("");

  function refresh() {
    const fresh = loadApps();
    setApps(fresh);
  }

  function updateStatus(id, status) {
    const updated = apps.map(a => a.id === id ? { ...a, status, managerNote: note } : a);
    saveApps(updated);
    setApps(updated);
    setSelected(updated.find(a => a.id === id));
  }

  const filtered = apps.filter(a => {
    const matchFilter = filter === "All" || a.status === filter;
    const matchSearch = !search || a.fullName?.toLowerCase().includes(search.toLowerCase()) || a.id?.includes(search);
    return matchFilter && matchSearch;
  });

  const counts = { All: apps.length, Pending: 0, Approved: 0, Rejected: 0 };
  apps.forEach(a => counts[a.status] = (counts[a.status] || 0) + 1);

  return (
    <div style={{ minHeight: "100vh", background: "#f1f8e9", display: "flex", flexDirection: "column" }}>
      {/* Top bar */}
      <div style={{ background: "#1b5e20", padding: "14px 24px", display: "flex", alignItems: "center", gap: 16, flexWrap: "wrap" }}>
        <Logo size={40}/>
        <div style={{ flex: 1 }}>
          <div style={{ color: "#fff", fontWeight: 900, fontFamily: "Georgia,serif", fontSize: 16 }}>NUVI Manager Dashboard</div>
          <div style={{ color: "#a5d6a7", fontSize: 11 }}>Application Management Portal</div>
        </div>
        <button onClick={onExit} style={{ padding: "8px 18px", background: "rgba(255,255,255,0.1)", color: "#fff", border: "1px solid rgba(255,255,255,0.3)", borderRadius: 8, cursor: "pointer", fontSize: 12 }}>
          🚪 Exit
        </button>
      </div>

      {/* Stats */}
      <div style={{ display: "flex", gap: 12, padding: "16px 20px", flexWrap: "wrap" }}>
        {["All","Pending","Approved","Rejected"].map(s => (
          <button key={s} onClick={() => setFilter(s)}
            style={{ flex: "1 1 100px", padding: "12px 8px", borderRadius: 10, border: `2px solid ${filter===s?"#2e7d32":"#c8e6c9"}`,
              background: filter===s?"#2e7d32":"#fff", color: filter===s?"#fff":"#2e7d32",
              fontWeight: 700, cursor: "pointer", fontSize: 13 }}>
            {s}<br/><span style={{ fontSize: 22, fontFamily: "Georgia,serif" }}>{counts[s] || 0}</span>
          </button>
        ))}
      </div>

      {/* Search */}
      <div style={{ padding: "0 20px 12px" }}>
        <input style={{ ...inputStyle, background: "#fff" }} placeholder="🔍 Search by name or application ID…"
          value={search} onChange={e => setSearch(e.target.value)}/>
      </div>

      <div style={{ display: "flex", flex: 1, gap: 0, overflow: "hidden", flexWrap: "wrap" }}>
        {/* List */}
        <div style={{ flex: "1 1 280px", overflowY: "auto", maxHeight: "70vh", padding: "0 12px 20px" }}>
          {filtered.length === 0 && (
            <div style={{ textAlign: "center", color: "#888", padding: 40 }}>No applications found</div>
          )}
          {filtered.map(a => (
            <div key={a.id} onClick={() => { setSelected(a); setNote(a.managerNote || ""); }}
              style={{ background: selected?.id === a.id ? "#e8f5e9" : "#fff", border: `2px solid ${selected?.id===a.id?"#2e7d32":"#e0e0e0"}`,
                borderRadius: 12, padding: "14px 16px", marginBottom: 10, cursor: "pointer", transition: "all 0.2s" }}>
              <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start", marginBottom: 6 }}>
                <div style={{ fontWeight: 700, color: "#1b5e20", fontSize: 14 }}>{a.fullName}</div>
                <Badge status={a.status}/>
              </div>
              <div style={{ fontSize: 12, color: "#777", fontFamily: "monospace" }}>{a.id}</div>
              <div style={{ fontSize: 12, color: "#555", marginTop: 4 }}>
                🌾 {a.cropType} — {a.empowermentType}
              </div>
              <div style={{ fontSize: 11, color: "#999", marginTop: 2 }}>
                📅 {new Date(a.submittedAt).toLocaleDateString()}
              </div>
            </div>
          ))}
        </div>

        {/* Detail Panel */}
        <div style={{ flex: "1 1 320px", padding: "0 16px 20px", overflowY: "auto", maxHeight: "70vh" }}>
          {!selected ? (
            <div style={{ textAlign: "center", color: "#aaa", padding: "60px 20px" }}>
              <div style={{ fontSize: 48, marginBottom: 10 }}>👈</div>
              Select an application to review
            </div>
          ) : (
            <div style={{ background: "#fff", borderRadius: 14, padding: "22px", boxShadow: "0 2px 16px rgba(0,0,0,0.07)" }}>
              <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 16 }}>
                <h3 style={{ margin: 0, color: "#1b5e20", fontFamily: "Georgia,serif" }}>{selected.fullName}</h3>
                <Badge status={selected.status}/>
              </div>

              <Section title="📋 Application">
                <Row label="ID" val={selected.id}/>
                <Row label="Submitted" val={new Date(selected.submittedAt).toLocaleString()}/>
                <Row label="Crop" val={selected.cropType}/>
                <Row label="Empowerment" val={selected.empowermentType}/>
                <Row label="Repayment" val={selected.repayment}/>
              </Section>

              <Section title="👤 Applicant">
                <Row label="Phone" val={selected.phone}/>
                <Row label="WhatsApp" val={selected.whatsapp}/>
                <Row label="Email" val={selected.email || "—"}/>
                <Row label="DOB" val={selected.dob}/>
                <Row label="NIN" val={selected.nin}/>
                <Row label="BVN" val={selected.bvn}/>
                <Row label="Address" val={selected.address}/>
              </Section>

              <Section title="🤝 Guarantor">
                <Row label="Name" val={selected.guarantorName}/>
                <Row label="Phone" val={selected.guarantorPhone}/>
                <Row label="NIN" val={selected.guarantorNin}/>
                <Row label="Address" val={selected.guarantorAddress}/>
              </Section>

              {selected.status === "Pending" && (
                <>
                  <div style={{ marginBottom: 14 }}>
                    <label style={{ fontWeight: 700, fontSize: 12, color: "#555", display: "block", marginBottom: 6 }}>MANAGER NOTE (optional)</label>
                    <textarea style={{ ...inputStyle, minHeight: 70, resize: "vertical" }} value={note} onChange={e => setNote(e.target.value)} placeholder="Add a note to send to applicant…"/>
                  </div>
                  <div style={{ display: "flex", gap: 10 }}>
                    <button onClick={() => updateStatus(selected.id, "Approved")}
                      style={{ flex: 1, padding: "13px", background: "#2e7d32", color: "#fff", border: "none", borderRadius: 10, fontWeight: 800, cursor: "pointer", fontSize: 14 }}>
                      ✅ APPROVE
                    </button>
                    <button onClick={() => updateStatus(selected.id, "Rejected")}
                      style={{ flex: 1, padding: "13px", background: "#c62828", color: "#fff", border: "none", borderRadius: 10, fontWeight: 800, cursor: "pointer", fontSize: 14 }}>
                      ❌ REJECT
                    </button>
                  </div>
                </>
              )}

              {selected.status !== "Pending" && (
                <div style={{ marginTop: 10 }}>
                  {selected.managerNote && (
                    <div style={{ background: "#f5f5f5", borderRadius: 8, padding: "10px 14px", fontSize: 13, color: "#555", marginBottom: 12 }}>
                      <strong>Manager Note:</strong> {selected.managerNote}
                    </div>
                  )}
                  <button onClick={() => updateStatus(selected.id, "Pending")}
                    style={{ width: "100%", padding: "11px", background: "#fff8e1", color: "#f57f17", border: "1.5px solid #ffe082", borderRadius: 10, fontWeight: 700, cursor: "pointer", fontSize: 13 }}>
                    ↩ Reset to Pending
                  </button>
                </div>
              )}
            </div>
          )}
        </div>
      </div>
    </div>
  );
}

function Section({ title, children }) {
  return (
    <div style={{ marginBottom: 18 }}>
      <div style={{ fontWeight: 700, fontSize: 12, color: "#1b5e20", letterSpacing: 1, textTransform: "uppercase", marginBottom: 8, borderBottom: "1.5px solid #e8f5e9", paddingBottom: 4 }}>{title}</div>
      {children}
    </div>
  );
}
function Row({ label, val }) {
  return (
    <div style={{ display: "flex", gap: 8, marginBottom: 5, fontSize: 13 }}>
      <span style={{ color: "#888", minWidth: 90, flexShrink: 0 }}>{label}:</span>
      <span style={{ color: "#222", fontWeight: 500, wordBreak: "break-word" }}>{val}</span>
    </div>
  );
}

// ═══════════════════════════════════════════════════════════════════════════
//  MANAGER LOGIN
// ═══════════════════════════════════════════════════════════════════════════
function ManagerLogin({ onLogin }) {
  const [pw, setPw] = useState("");
  const [err, setErr] = useState(false);

  function attempt() {
    if (pw === MANAGER_PASSWORD) { onLogin(); }
    else { setErr(true); setTimeout(() => setErr(false), 2000); }
  }

  return (
    <div style={{ minHeight: "100vh", background: "linear-gradient(135deg,#1b5e20,#2e7d32,#388e3c)", display: "flex", alignItems: "center", justifyContent: "center", padding: 20 }}>
      <div style={{ background: "#fff", borderRadius: 20, padding: "40px 32px", maxWidth: 380, width: "100%", textAlign: "center", boxShadow: "0 20px 60px rgba(0,0,0,0.3)" }}>
        <Logo size={64}/>
        <h2 style={{ color: "#1b5e20", fontFamily: "Georgia,serif", margin: "16px 0 4px" }}>Manager Portal</h2>
        <p style={{ color: "#888", fontSize: 13, marginBottom: 24 }}>NUVI Farmers Progress Company</p>
        <input type="password" style={{ ...inputStyle, marginBottom: 14, border: err ? "2px solid #c62828" : "1.5px solid #c8e6c9" }}
          placeholder="Enter manager password" value={pw} onChange={e => setPw(e.target.value)}
          onKeyDown={e => e.key === "Enter" && attempt()}/>
        {err && <div style={{ color: "#c62828", fontSize: 13, marginBottom: 10 }}>Incorrect password. Try again.</div>}
        <button onClick={attempt} style={{ width: "100%", padding: "13px", background: "#1b5e20", color: "#fff", border: "none", borderRadius: 10, fontWeight: 800, cursor: "pointer", fontSize: 15 }}>
          🔐 LOGIN
        </button>
        <p style={{ marginTop: 16, fontSize: 11, color: "#bbb" }}>Default password: nuvi2026</p>
      </div>
    </div>
  );
}

// ═══════════════════════════════════════════════════════════════════════════
//  ROOT
// ═══════════════════════════════════════════════════════════════════════════
export default function App() {
  const [view, setView] = useState("home"); // home | form | success | managerLogin | dashboard
  const [lastApp, setLastApp] = useState(null);
  const [managerAuth, setManagerAuth] = useState(false);

  if (view === "form") return <CustomerForm onDone={app => { setLastApp(app); setView("success"); }}/>;
  if (view === "success") return <SuccessScreen app={lastApp} onReset={() => setView("form")}/>;
  if (view === "managerLogin") {
    if (managerAuth) return <ManagerDashboard onExit={() => { setManagerAuth(false); setView("home"); }}/>;
    return <ManagerLogin onLogin={() => { setManagerAuth(true); setView("dashboard"); }}/>;
  }
  if (view === "dashboard" && managerAuth) return <ManagerDashboard onExit={() => { setManagerAuth(false); setView("home"); }}/>;

  // HOME LANDING PAGE
  return (
    <div style={{ minHeight: "100vh", background: "linear-gradient(160deg,#e8f5e9 0%,#f9fbe7 50%,#fff8e1 100%)", position: "relative", overflow: "hidden" }}>
      <LeafDeco style={{ position: "absolute", top: -10, right: -10, width: 200 }}/>
      <LeafDeco style={{ position: "absolute", bottom: 40, left: -20, width: 160, transform: "rotate(180deg)" }}/>

      {/* Nav */}
      <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", padding: "16px 24px" }}>
        <div style={{ display: "flex", alignItems: "center", gap: 10 }}>
          <Logo size={40}/>
          <div>
            <div style={{ fontFamily: "Georgia,serif", fontWeight: 900, color: "#1b5e20", fontSize: 14, lineHeight: 1.1 }}>NUVI FARMERS</div>
            <div style={{ fontSize: 9, color: "#888", letterSpacing: 1 }}>PROGRESS CO. INTL. LTD.</div>
          </div>
        </div>
        <button onClick={() => setView("managerLogin")} style={{ padding: "8px 16px", background: "#1b5e20", color: "#fff", border: "none", borderRadius: 8, fontWeight: 700, cursor: "pointer", fontSize: 12 }}>
          Manager Login
        </button>
      </div>

      {/* Hero */}
      <div style={{ textAlign: "center", padding: "40px 24px 20px" }}>
        <div style={{ fontSize: 56, marginBottom: 10 }}>🌾</div>
        <h1 style={{ fontFamily: "Georgia,serif", color: "#1b5e20", fontSize: "clamp(22px,5vw,38px)", margin: "0 0 12px", lineHeight: 1.2 }}>
          Farmer Empowerment<br/>Application Portal
        </h1>
        <p style={{ color: "#555", maxWidth: 480, margin: "0 auto 10px", fontSize: 15, lineHeight: 1.7 }}>
          <em>"In honesty and harmony we transcend"</em>
        </p>
        <p style={{ color: "#666", maxWidth: 520, margin: "0 auto 32px", fontSize: 14, lineHeight: 1.7 }}>
          Apply for agricultural empowerment support — receive 1 carton of chemicals or its cash equivalent to grow your farm and repay with your harvest.
        </p>
        <button onClick={() => setView("form")} style={{
          padding: "18px 48px", background: "#2e7d32", color: "#fff", border: "none",
          borderRadius: 50, fontWeight: 900, fontSize: 16, cursor: "pointer",
          boxShadow: "0 8px 24px rgba(46,125,50,0.35)", letterSpacing: 1,
          transition: "transform 0.2s"
        }}
          onMouseOver={e => e.target.style.transform="scale(1.04)"}
          onMouseOut={e => e.target.style.transform="scale(1)"}>
          APPLY NOW →
        </button>
      </div>

      {/* Info cards */}
      <div style={{ display: "flex", gap: 16, flexWrap: "wrap", maxWidth: 700, margin: "0 auto", padding: "20px 20px 60px" }}>
        {[
          { icon: "🧪", title: "What You Receive", body: "1 carton of farming chemical OR its cash equivalent to support your farming season." },
          { icon: "📅", title: "Distribution Period", body: "Applications open 15th May to 15th June every year. Don't miss the window!" },
          { icon: "🌽", title: "Repayment", body: "Repay with bags of your harvested crop on or before 20th December." },
          { icon: "📋", title: "Requirements", body: "Valid NIN, BVN, passport photo, guarantor details, and you must be 18+ years old." },
        ].map(c => (
          <div key={c.title} style={{ flex: "1 1 200px", background: "#fff", borderRadius: 14, padding: "20px 18px", boxShadow: "0 2px 12px rgba(0,0,0,0.07)", border: "1px solid #e8f5e9" }}>
            <div style={{ fontSize: 28, marginBottom: 8 }}>{c.icon}</div>
            <div style={{ fontWeight: 800, color: "#1b5e20", marginBottom: 6, fontSize: 14 }}>{c.title}</div>
            <div style={{ fontSize: 13, color: "#666", lineHeight: 1.6 }}>{c.body}</div>
          </div>
        ))}
      </div>

      <div style={{ textAlign: "center", padding: "0 20px 30px", fontSize: 12, color: "#aaa" }}>
        © 2026 NUVI Farmers Progress Company International Ltd. · All Rights Reserved
      </div>
    </div>
  );
}
