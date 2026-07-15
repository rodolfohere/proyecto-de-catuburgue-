import { useState, useEffect } from "react";

// ══════════════════════════════════════════════════════════════
//  ⚙️  CONFIGURACIÓN — Edita antes de publicar
// ══════════════════════════════════════════════════════════════
const OWNER_CONFIG = {
  whatsapp:  "584122060825",
  ntfyTopic: "cactus-burger-charallave",
  pin:       "1234",
};

// ══════════════════════════════════════════════════════════════
//  🌟 CLUB CACTUS — Tabla de recompensas
// ══════════════════════════════════════════════════════════════
const REWARDS = [
  { pts:50,  label:"Refresco Gratis",           emoji:"🥤", color:"#3B82F6" },
  { pts:80,  label:"Papas Fritas Gratis",       emoji:"🍟", color:"#F59E0B" },
  { pts:120, label:"Hamburguesa Simple Gratis", emoji:"🍔", color:"#10B981" },
  { pts:180, label:"Combo con Descuento",       emoji:"🎊", color:"#8B5CF6" },
];

// ══════════════════════════════════════════════════════════════
//  🍔 MENÚ
// ══════════════════════════════════════════════════════════════
const MENU_CATEGORIES = [
  {
    id:"combos", label:"Combos", emoji:"🎁",
    items:[
      { id:"c1", emoji:"🌵", name:"Combo Cactus",  desc:"2 Hamburguesas Simple + 2 Papelón con Limón",      price:5.50, isCombo:true },
      { id:"c2", emoji:"🔥", name:"Combo Fuego",   desc:"2 Hamburguesas de Carne + 2 Refrescos",            price:8.00, isCombo:true },
      { id:"c3", emoji:"🌭", name:"Combo Perrito", desc:"2 Perros Cal. Especial + 2 Salchipapas Sencilla",  price:9.00, isCombo:true },
      { id:"c4", emoji:"⚡", name:"Combo Rápido",  desc:"1 Hamburguesa Simple + 1 Malta + 1 Papas Fritas",  price:4.00, isCombo:true },
    ]
  },
  {
    id:"hamburguesas", label:"Hamburguesas", emoji:"🍔",
    items:[
      { id:1,  emoji:"🥩", name:"Hamburguesa de Carne",    desc:"Jugosa carne de res a la parrilla",        price:3.50 },
      { id:2,  emoji:"🍗", name:"Hamburguesa de Pollo",    desc:"Pollo crujiente con aderezos de la casa",  price:3.00 },
      { id:3,  emoji:"🐷", name:"Hamburguesa de Cerdo",    desc:"Cerdo tierno con salsa especial",          price:3.50 },
      { id:4,  emoji:"🍔", name:"Hamburguesa Simple",      desc:"Clásica y deliciosa, sin complicaciones",  price:2.00 },
    ]
  },
  {
    id:"salchipapas", label:"Salchipapas", emoji:"🍟",
    items:[
      { id:5, emoji:"🍟", name:"Salchipapas Sencilla", desc:"Papas doradas con salchicha",            price:2.50 },
      { id:6, emoji:"⭐", name:"Salchipapas Especial", desc:"Con queso, salsas y toppings premium",   price:3.50 },
    ]
  },
  {
    id:"perros", label:"Perros Calientes", emoji:"🌭",
    items:[
      { id:7, emoji:"🌭", name:"Perro Caliente Simple",   desc:"Clásico con mostaza y kétchup",          price:2.00 },
      { id:8, emoji:"🌟", name:"Perro Caliente Especial", desc:"Con topping completo y salsa especial",  price:3.00 },
    ]
  },
  {
    id:"bebidas", label:"Bebidas", emoji:"🥤",
    items:[
      { id:9,  emoji:"🍺", name:"Malta",             desc:"Refrescante malta bien fría",          price:1.00 },
      { id:10, emoji:"🍋", name:"Papelón con Limón", desc:"Bebida tradicional venezolana",        price:1.00 },
      { id:11, emoji:"🥤", name:"Refresco",          desc:"Tu sabor preferido bien helado",       price:1.00 },
    ]
  },
  {
    id:"extras", label:"Extras", emoji:"✨",
    items:[
      { id:12, emoji:"🍟", name:"Ración de Papas Fritas", desc:"Crujientes papas doradas al momento", price:1.50 },
    ]
  },
];

const PAYMENTS = [
  { id:"efectivo",      label:"Efectivo",     icon:"💵" },
  { id:"pago-movil",    label:"Pago Móvil",   icon:"📱" },
  { id:"transferencia", label:"Transferencia", icon:"🏦" },
  { id:"zelle",         label:"Zelle",        icon:"💸" },
  { id:"binance",       label:"Binance",      icon:"🔶" },
];

// ══════════════════════════════════════════════════════════════
//  🎨 COLORES
// ══════════════════════════════════════════════════════════════
const C = {
  bg:"#0C0C10", surface:"#16161D", hi:"#1E1E28",
  gold:"#F5A623", green:"#2EBD6B", red:"#E74C3C", blue:"#3B82F6", purple:"#8B5CF6",
  white:"#F5F5F7", muted:"#6B7280", light:"#A1A1AA", border:"#26262E",
};

// ══════════════════════════════════════════════════════════════
//  🔧 UTILIDADES
// ══════════════════════════════════════════════════════════════
const fmt  = (n) => `$${Number(n).toFixed(2)}`;
const pad  = (n) => String(n).padStart(3,"0");
const dkey = ()  => new Date().toISOString().split("T")[0];
const tstr = ()  => new Date().toLocaleTimeString("es-VE",{hour:"2-digit",minute:"2-digit"});

// LocalStorage helpers
const lsGet = (key, def=null) => {
  try { const v=localStorage.getItem(key); return v ? JSON.parse(v) : def; } catch { return def; }
};
const lsSet = (key, val) => { try { localStorage.setItem(key,JSON.stringify(val)); } catch {} };

// Club helpers
const cleanPhone = (p) => p.replace(/\D/g,"");
const getClub    = (phone) => lsGet(`cb-club-${cleanPhone(phone)}`, null);
const saveClub   = (phone, data) => lsSet(`cb-club-${cleanPhone(phone)}`, data);
const getMembers = () => lsGet("cb-club-members", []);
const addMember  = (phone) => {
  const c = cleanPhone(phone); if (!c) return;
  const list = getMembers();
  if (!list.includes(c)) lsSet("cb-club-members", [...list, c]);
};
const awardPoints = (phone, name, amount, orderId) => {
  if (!cleanPhone(phone)) return null;
  const pts = Math.floor(amount);
  const ex  = getClub(phone) || { name:"", phone:cleanPhone(phone), points:0, history:[] };
  const updated = {
    ...ex,
    name: name || ex.name,
    points: ex.points + pts,
    history: [{ date:dkey(), orderId, pts, total:amount }, ...ex.history].slice(0,40),
  };
  saveClub(phone, updated);
  addMember(phone);
  return updated;
};
const deductPts = (phone, pts) => {
  const ex = getClub(phone); if (!ex) return null;
  const updated = {
    ...ex,
    points: Math.max(0, ex.points - pts),
    history: [{ date:dkey(), orderId:"CANJE", pts:-pts, total:0 }, ...ex.history].slice(0,40),
  };
  saveClub(phone, updated);
  return updated;
};
const nextReward = (pts) => REWARDS.find(r => r.pts > pts) || null;

// ══════════════════════════════════════════════════════════════
//  🧩 COMPONENTE PRINCIPAL
// ══════════════════════════════════════════════════════════════
export default function CactusBurgerMenu() {

  // ── Menú ──────────────────────────────────────────────────
  const [activeCat, setActiveCat] = useState(0);
  const [cart,      setCart]      = useState({});
  const [cartOpen,  setCartOpen]  = useState(false);
  const [popped,    setPopped]    = useState(null);

  // ── Flujo de pedido ───────────────────────────────────────
  // status: idle | checkout | sending | sent
  const [status,     setStatus]      = useState("idle");
  const [orderNum,   setOrderNum]    = useState(null);
  const [savedTotal, setSavedTotal]  = useState(0);
  const [savedItems, setSavedItems]  = useState([]);
  const [savedCust,  setSavedCust]   = useState(null);
  const [earnedPts,  setEarnedPts]   = useState(null);

  // ── Formulario de cliente ─────────────────────────────────
  const [cust, setCust] = useState({
    name:"", address:"", notes:"",
    payment:"efectivo", delivery:"retiro", clubPhone:"",
  });
  const [errors, setErrors] = useState({});

  // ── Club Cactus (pantalla pública) ────────────────────────
  const [clubOpen,     setClubOpen]     = useState(false);
  const [clubSearch,   setClubSearch]   = useState("");
  const [clubResult,   setClubResult]   = useState(null);
  const [clubNotFound, setClubNotFound] = useState(false);

  // ── Configuración ─────────────────────────────────────────
  const [cfgWA,   setCfgWA]   = useState(OWNER_CONFIG.whatsapp);
  const [cfgNtfy, setCfgNtfy] = useState(OWNER_CONFIG.ntfyTopic);
  const [cfgPin,  setCfgPin]  = useState(OWNER_CONFIG.pin);

  // ── Panel Dueño ───────────────────────────────────────────
  const [logoTaps,  setLogoTaps]  = useState(0);
  const [showPin,   setShowPin]   = useState(false);
  const [pinInput,  setPinInput]  = useState("");
  const [pinErr,    setPinErr]    = useState(false);
  const [ownerOpen, setOwnerOpen] = useState(false);
  const [ownerTab,  setOwnerTab]  = useState("today");
  const [orders,    setOrders]    = useState([]);
  const [selOrder,  setSelOrder]  = useState(null);
  const [loading,   setLoading]   = useState(false);
  const [cfgSaved,  setCfgSaved]  = useState(false);
  const [newWA,     setNewWA]     = useState("");
  const [newNtfy,   setNewNtfy]   = useState("");
  const [newPin,    setNewPin]    = useState("");
  // Club panel
  const [members,   setMembers]   = useState([]);
  const [selMember, setSelMember] = useState(null);
  const [deductAmt, setDeductAmt] = useState("");
  const [deductOk,  setDeductOk]  = useState(false);

  // Cargar config guardada al montar
  useEffect(() => {
    const cfg = lsGet("cb-config");
    if (cfg) {
      if (cfg.wa)   setCfgWA(cfg.wa);
      if (cfg.ntfy) setCfgNtfy(cfg.ntfy);
      if (cfg.pin)  setCfgPin(cfg.pin);
    }
  }, []);

  // ── Carrito ───────────────────────────────────────────────
  const addItem = (item) => {
    setCart(p => ({...p, [item.id]:{...item, qty:(p[item.id]?.qty||0)+1}}));
    setPopped(item.id);
    setTimeout(() => setPopped(null), 380);
  };
  const remItem = (id) => setCart(p => {
    const n={...p};
    if (n[id]?.qty>1) n[id]={...n[id],qty:n[id].qty-1}; else delete n[id];
    return n;
  });

  const items = Object.values(cart);
  const count = items.reduce((s,i) => s+i.qty, 0);
  const total = items.reduce((s,i) => s+i.price*i.qty, 0);

  // ── Mensaje WhatsApp / ntfy ───────────────────────────────
  const buildMsg = (num, its, tot, c) => {
    const lines    = its.map(i=>`${i.qty}x ${i.name}`).join("\n");
    const notesTxt = c.notes   ? `\n📝 ${c.notes}` : "";
    const payLbl   = PAYMENTS.find(p=>p.id===c.payment);
    const delvTxt  = c.delivery==="delivery"
      ? `🛵 *DELIVERY*\n📍 ${c.address}`
      : `🏪 *RETIRO EN LOCAL*`;
    return `🌵 *CACTUS BURGER* — Pedido #${pad(num)}\n👤 ${c.name}\n${delvTxt}\n${payLbl?.icon||""} ${payLbl?.label||c.payment}\n\n${lines}${notesTxt}\n\n💰 *Total: ${fmt(tot)}*\n⏰ ${tstr()}`;
  };

  // ── Confirmar pedido ──────────────────────────────────────
  const handleConfirm = async () => {
    const errs = {};
    if (!cust.name.trim())                                   errs.name    = true;
    if (cust.delivery==="delivery" && !cust.address.trim()) errs.address = true;
    if (!cust.payment)                                       errs.payment = true;
    setErrors(errs);
    if (Object.keys(errs).length) return;

    setStatus("sending");
    const num  = Math.floor(Math.random()*900)+100;
    const snap = [...items];
    const tot  = total;

    setOrderNum(num);
    setSavedTotal(tot);
    setSavedItems(snap);
    setSavedCust({...cust});

    const order = {
      id:num, date:dkey(), timestamp:new Date().toISOString(),
      items:snap.map(i=>({id:i.id,name:i.name,qty:i.qty,price:i.price})),
      notes:cust.notes, customer:{...cust}, total:tot,
    };

    // 1. Guardar en localStorage
    try {
      const prev = lsGet("cb-orders",[]);
      lsSet("cb-orders",[...prev, order]);
    } catch {}

    // 2. Puntos Club Cactus
    if (cust.clubPhone) {
      const updated = awardPoints(cust.clubPhone, cust.name, tot, num);
      if (updated) setEarnedPts({ pts:Math.floor(tot), total:updated.points });
    }

    // 3. Notificación ntfy.sh
    try {
      await fetch(`https://ntfy.sh/${cfgNtfy}`, {
        method:"POST",
        headers:{"Title":"🌵 Nuevo Pedido CACTUS BURGER","Priority":"high","Tags":"hamburger,money_with_wings"},
        body: buildMsg(num, snap, tot, cust).replace(/\*/g,""),
      });
    } catch {}

    setCart({});
    setCust({name:"",address:"",notes:"",payment:"efectivo",delivery:"retiro",clubPhone:""});
    setStatus("sent");
  };

  const handleWhatsApp = () => {
    if (!savedCust) return;
    window.open(`https://wa.me/${cfgWA}?text=${encodeURIComponent(buildMsg(orderNum,savedItems,savedTotal,savedCust))}`, "_blank");
  };

  const resetOrder = () => { setStatus("idle"); setOrderNum(null); setEarnedPts(null); };

  // ── Club: buscar miembro ──────────────────────────────────
  const lookupClub = () => {
    const data = getClub(clubSearch);
    if (data) { setClubResult(data); setClubNotFound(false); }
    else      { setClubResult(null); setClubNotFound(true); }
  };

  // ── Owner: PIN ────────────────────────────────────────────
  const onLogoTap = () => {
    const n = logoTaps+1; setLogoTaps(n);
    if (n>=5) { setLogoTaps(0); setShowPin(true); }
  };
  const tapKey = (k) => {
    if (k==="⌫") { setPinInput(p=>p.slice(0,-1)); return; }
    if (pinInput.length>=4) return;
    const next = pinInput+k; setPinInput(next);
    if (next.length===4) setTimeout(()=>{
      if (next===cfgPin) {
        setShowPin(false); setPinInput(""); setPinErr(false);
        setOwnerOpen(true); loadOrders(); loadMembers();
      } else { setPinErr(true); setTimeout(()=>{setPinErr(false);setPinInput("");},900); }
    }, 200);
  };

  // ── Owner: datos ──────────────────────────────────────────
  const loadOrders = () => {
    setLoading(true);
    setOrders(lsGet("cb-orders",[]));
    setLoading(false);
  };
  const loadMembers = () => {
    const phones = getMembers();
    setMembers(phones.map(p => getClub(p)).filter(Boolean));
  };
  const clearOrders = () => { lsSet("cb-orders",[]); setOrders([]); };
  const saveConfig  = () => {
    const cfg = { wa:newWA||cfgWA, ntfy:newNtfy||cfgNtfy, pin:newPin||cfgPin };
    lsSet("cb-config", cfg);
    if (newWA)   setCfgWA(newWA);
    if (newNtfy) setCfgNtfy(newNtfy);
    if (newPin)  setCfgPin(newPin);
    setNewWA(""); setNewNtfy(""); setNewPin("");
    setCfgSaved(true); setTimeout(()=>setCfgSaved(false),2200);
  };
  const handleDeduct = () => {
    if (!selMember || !deductAmt) return;
    const updated = deductPts(selMember.phone, parseInt(deductAmt));
    if (updated) {
      setSelMember(updated);
      setMembers(m => m.map(x => x.phone===updated.phone ? updated : x));
      setDeductOk(true); setDeductAmt("");
      setTimeout(()=>setDeductOk(false), 2000);
    }
  };

  // Stats
  const todayOrders  = orders.filter(o=>o.date===dkey());
  const todayRevenue = todayOrders.reduce((s,o)=>s+o.total,0);
  const totalRevenue = orders.reduce((s,o)=>s+o.total,0);
  const itemCounts   = {};
  orders.forEach(o=>o.items?.forEach(i=>{itemCounts[i.name]=(itemCounts[i.name]||0)+i.qty;}));
  const topItems = Object.entries(itemCounts).sort((a,b)=>b[1]-a[1]).slice(0,5);
  const cur = MENU_CATEGORIES[activeCat];

  // ════════════════════════════════════════════════════════════
  //  CSS compartido
  // ════════════════════════════════════════════════════════════
  const STYLES = `
    @import url('https://fonts.googleapis.com/css2?family=Bebas+Neue&family=Nunito:wght@400;600;700;800;900&display=swap');
    *{box-sizing:border-box;margin:0;padding:0}
    ::-webkit-scrollbar{width:4px}::-webkit-scrollbar-track{background:transparent}::-webkit-scrollbar-thumb{background:#26262E;border-radius:4px}
    @keyframes pop{0%{transform:scale(1)}45%{transform:scale(1.28)}100%{transform:scale(1)}}
    @keyframes slideUp{from{transform:translateY(100%);opacity:0}to{transform:translateY(0);opacity:1}}
    @keyframes fadeInUp{from{transform:translateY(12px);opacity:0}to{transform:translateY(0);opacity:1}}
    @keyframes shake{0%,100%{transform:translateX(0)}20%{transform:translateX(-8px)}40%{transform:translateX(8px)}60%{transform:translateX(-5px)}80%{transform:translateX(5px)}}
    @keyframes pulse{0%,100%{transform:scale(1)}50%{transform:scale(1.08)}}
    @keyframes sparkle{0%,100%{opacity:1;transform:scale(1)}50%{opacity:.7;transform:scale(1.05)}}
    .item-card{transition:transform .2s,background-color .2s}
    .item-card:hover{transform:translateY(-2px);background-color:#1E1E28!important}
    .add-btn{transition:transform .15s,background-color .15s;cursor:pointer}
    .add-btn:hover{background-color:#E8950F!important}
    .add-btn.pop{animation:pop .38s ease}
    .tab{transition:color .2s;cursor:pointer}
    .tab:hover{color:#F5A623!important}
    .pkey{transition:background-color .1s,transform .1s;cursor:pointer}
    .pkey:hover{background-color:#2A2A36!important;transform:scale(1.05)}
    .pkey:active{transform:scale(.94)}
    .shake{animation:shake .5s ease}
    .pay-btn{transition:all .15s;cursor:pointer}
    .pay-btn:hover{border-color:#F5A623!important}
    .del-btn{transition:all .15s;cursor:pointer}
    input,textarea{outline:none}
  `;

  // ════════════════════════════════════════════════════════════
  //  PANTALLA: PEDIDO CONFIRMADO
  // ════════════════════════════════════════════════════════════
  if (status==="sent") return (
    <div style={{minHeight:"100vh",background:C.bg,display:"flex",flexDirection:"column",alignItems:"center",justifyContent:"center",padding:24,fontFamily:"'Nunito',sans-serif",color:C.white}}>
      <style>{STYLES}</style>
      <div style={{textAlign:"center",maxWidth:380,width:"100%",animation:"fadeInUp .4s ease"}}>
        <div style={{fontSize:64,marginBottom:16,animation:"pulse 1.2s ease infinite"}}>✅</div>
        <h1 style={{fontFamily:"'Bebas Neue'",fontSize:32,color:C.green,letterSpacing:2,marginBottom:6}}>¡Pedido Enviado!</h1>
        <p style={{color:C.light,fontSize:14,marginBottom:4}}>
          Pedido <strong style={{color:C.gold}}>#{pad(orderNum)}</strong> · Total: <strong style={{color:C.gold}}>{fmt(savedTotal)}</strong>
        </p>
        {savedCust && (
          <p style={{color:C.muted,fontSize:13,marginBottom:16}}>
            {savedCust.delivery==="delivery" ? "🛵 Delivery" : "🏪 Retiro en local"} · {PAYMENTS.find(p=>p.id===savedCust.payment)?.label}
          </p>
        )}
        {/* Puntos ganados */}
        {earnedPts && (
          <div style={{background:"linear-gradient(135deg,rgba(245,166,35,.18),rgba(245,166,35,.06))",border:`1px solid rgba(245,166,35,.4)`,borderRadius:16,padding:"14px 18px",marginBottom:20,animation:"sparkle 1.5s ease infinite"}}>
            <p style={{fontFamily:"'Bebas Neue'",fontSize:22,color:C.gold,letterSpacing:1}}>🌵 +{earnedPts.pts} puntos Club Cactus</p>
            <p style={{color:C.light,fontSize:13}}>Tu saldo: <strong style={{color:C.gold}}>{earnedPts.total} puntos</strong></p>
            {nextReward(earnedPts.total) && <p style={{color:C.muted,fontSize:12,marginTop:4}}>Próxima recompensa en {nextReward(earnedPts.total).pts - earnedPts.total} pts más · {nextReward(earnedPts.total).label}</p>}
          </div>
        )}
        <p style={{color:C.muted,fontSize:13,marginBottom:24,lineHeight:1.7}}>Confirma tu pedido por WhatsApp para mayor seguridad. 📲</p>
        <button onClick={handleWhatsApp} style={{width:"100%",padding:15,background:"#25D366",color:"#fff",border:"none",borderRadius:16,fontFamily:"'Nunito'",fontWeight:900,fontSize:16,cursor:"pointer",marginBottom:12,display:"flex",alignItems:"center",justifyContent:"center",gap:10}}>
          <span style={{fontSize:20}}>📲</span> Confirmar por WhatsApp
        </button>
        <button onClick={resetOrder} style={{width:"100%",padding:14,background:C.surface,color:C.light,border:`1px solid ${C.border}`,borderRadius:16,fontFamily:"'Nunito'",fontWeight:800,fontSize:15,cursor:"pointer"}}>
          Hacer otro pedido
        </button>
      </div>
    </div>
  );

  // ════════════════════════════════════════════════════════════
  //  PANTALLA: DATOS DEL CLIENTE (CHECKOUT)
  // ════════════════════════════════════════════════════════════
  if (status==="checkout" || status==="sending") return (
    <div style={{minHeight:"100vh",background:C.bg,color:C.white,fontFamily:"'Nunito',sans-serif",paddingBottom:100}}>
      <style>{STYLES}</style>
      {/* Header */}
      <div style={{background:C.surface,borderBottom:`1.5px solid ${C.border}`,padding:"14px 16px",position:"sticky",top:0,zIndex:10}}>
        <div style={{maxWidth:600,margin:"0 auto",display:"flex",alignItems:"center",gap:14}}>
          <button onClick={()=>setStatus("idle")} style={{background:"transparent",border:"none",color:C.gold,fontSize:22,cursor:"pointer",display:"flex",alignItems:"center",padding:4}}>←</button>
          <div>
            <h2 style={{fontFamily:"'Bebas Neue'",fontSize:22,color:C.gold,letterSpacing:2}}>DATOS DEL PEDIDO</h2>
            <p style={{color:C.muted,fontSize:12}}>Completa tu información</p>
          </div>
        </div>
      </div>

      <div style={{maxWidth:600,margin:"0 auto",padding:"16px 14px",display:"flex",flexDirection:"column",gap:14}}>
        {/* Resumen mini del carrito */}
        <div style={{background:C.surface,borderRadius:16,border:`1px solid ${C.border}`,padding:16}}>
          <p style={{fontWeight:800,fontSize:13,color:C.light,marginBottom:10}}>🧾 Tu pedido</p>
          {items.map(i => (
            <div key={i.id} style={{display:"flex",justifyContent:"space-between",marginBottom:6}}>
              <span style={{fontSize:13,color:C.white}}>{i.qty}x {i.name}</span>
              <span style={{fontSize:13,color:C.gold,fontWeight:700}}>{fmt(i.price*i.qty)}</span>
            </div>
          ))}
          <div style={{borderTop:`1px solid ${C.border}`,paddingTop:10,marginTop:6,display:"flex",justifyContent:"space-between"}}>
            <span style={{fontWeight:800,color:C.light}}>Total</span>
            <span style={{fontFamily:"'Bebas Neue'",fontSize:22,color:C.gold}}>{fmt(total)}</span>
          </div>
        </div>

        {/* Nombre */}
        <div>
          <label style={{fontSize:13,fontWeight:700,color:C.light,display:"block",marginBottom:8}}>
            👤 Tu nombre <span style={{color:C.red}}>*</span>
          </label>
          <input
            value={cust.name}
            onChange={e=>setCust(p=>({...p,name:e.target.value}))}
            placeholder="¿Cómo te llamamos?"
            style={{width:"100%",background:C.surface,border:`1.5px solid ${errors.name?C.red:C.border}`,borderRadius:12,padding:"12px 14px",color:C.white,fontFamily:"'Nunito'",fontSize:14,transition:"border-color .2s"}}
          />
          {errors.name && <p style={{color:C.red,fontSize:12,marginTop:4}}>El nombre es obligatorio</p>}
        </div>

        {/* Modo de entrega */}
        <div>
          <label style={{fontSize:13,fontWeight:700,color:C.light,display:"block",marginBottom:8}}>📦 Modo de entrega</label>
          <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10}}>
            {[["retiro","🏪","RETIRO EN LOCAL"],["delivery","🛵","DELIVERY"]].map(([id,ico,lbl])=>(
              <button key={id} className="del-btn" onClick={()=>setCust(p=>({...p,delivery:id}))} style={{padding:"14px 8px",background:cust.delivery===id?C.gold:C.surface,color:cust.delivery===id?"#0C0C10":C.light,border:`1.5px solid ${cust.delivery===id?C.gold:C.border}`,borderRadius:14,fontFamily:"'Nunito'",fontWeight:900,fontSize:13,display:"flex",flexDirection:"column",alignItems:"center",gap:6}}>
                <span style={{fontSize:24}}>{ico}</span>{lbl}
              </button>
            ))}
          </div>
        </div>

        {/* Dirección (solo delivery) */}
        {cust.delivery==="delivery" && (
          <div style={{animation:"fadeInUp .2s ease"}}>
            <label style={{fontSize:13,fontWeight:700,color:C.light,display:"block",marginBottom:8}}>
              📍 Dirección de entrega <span style={{color:C.red}}>*</span>
            </label>
            <textarea
              value={cust.address}
              onChange={e=>setCust(p=>({...p,address:e.target.value}))}
              placeholder="Calle, número, referencia..."
              rows={3}
              style={{width:"100%",background:C.surface,border:`1.5px solid ${errors.address?C.red:C.border}`,borderRadius:12,padding:"12px 14px",color:C.white,fontFamily:"'Nunito'",fontSize:14,resize:"none"}}
            />
            {errors.address && <p style={{color:C.red,fontSize:12,marginTop:4}}>La dirección es obligatoria para delivery</p>}
          </div>
        )}

        {/* Método de pago */}
        <div>
          <label style={{fontSize:13,fontWeight:700,color:C.light,display:"block",marginBottom:8}}>
            💳 Método de pago <span style={{color:C.red}}>*</span>
          </label>
          <div style={{display:"flex",flexWrap:"wrap",gap:8}}>
            {PAYMENTS.map(p=>(
              <button key={p.id} className="pay-btn" onClick={()=>setCust(c=>({...c,payment:p.id}))} style={{padding:"9px 14px",background:cust.payment===p.id?C.gold:C.surface,color:cust.payment===p.id?"#0C0C10":C.light,border:`1.5px solid ${cust.payment===p.id?C.gold:C.border}`,borderRadius:50,fontFamily:"'Nunito'",fontWeight:800,fontSize:13,display:"flex",alignItems:"center",gap:6}}>
                {p.icon} {p.label}
              </button>
            ))}
          </div>
        </div>

        {/* Notas */}
        <div>
          <label style={{fontSize:13,fontWeight:700,color:C.light,display:"block",marginBottom:8}}>📝 Notas especiales (opcional)</label>
          <textarea
            value={cust.notes}
            onChange={e=>setCust(p=>({...p,notes:e.target.value}))}
            placeholder="Sin cebolla · Extra salsa · Todo bien cocido..."
            rows={2}
            style={{width:"100%",background:C.surface,border:`1px solid ${C.border}`,borderRadius:12,padding:"11px 14px",color:C.white,fontFamily:"'Nunito'",fontSize:13,resize:"none"}}
          />
        </div>

        {/* Club Cactus */}
        <div style={{background:"linear-gradient(135deg,rgba(245,166,35,.12),rgba(245,166,35,.04))",border:`1.5px solid rgba(245,166,35,.35)`,borderRadius:16,padding:16}}>
          <div style={{display:"flex",alignItems:"center",gap:10,marginBottom:10}}>
            <span style={{fontSize:26}}>🌵</span>
            <div>
              <p style={{fontWeight:900,fontSize:14,color:C.gold}}>CLUB CACTUS</p>
              <p style={{color:C.muted,fontSize:12}}>Gana <strong style={{color:C.gold}}>{Math.floor(total)} puntos</strong> con este pedido · 1$ = 1 punto</p>
            </div>
          </div>
          <input
            value={cust.clubPhone}
            onChange={e=>setCust(p=>({...p,clubPhone:e.target.value}))}
            placeholder="Tu número de teléfono (opcional)"
            style={{width:"100%",background:C.bg,border:`1px solid rgba(245,166,35,.3)`,borderRadius:10,padding:"10px 12px",color:C.white,fontFamily:"'Nunito'",fontSize:13}}
          />
          {cust.clubPhone && (() => {
            const d = getClub(cust.clubPhone);
            const future = (d?.points||0) + Math.floor(total);
            const nr = nextReward(future);
            return (
              <p style={{color:C.gold,fontSize:12,marginTop:8}}>
                {d ? `Saldo actual: ${d.points} pts → pasarás a ${future} pts` : `¡Te registrarás con ${Math.floor(total)} puntos!`}
                {nr ? ` · ${nr.pts-future} pts para ${nr.label}` : " · ¡Premio disponible!"}
              </p>
            );
          })()}
        </div>

        {/* Botón confirmar */}
        <button
          onClick={handleConfirm}
          disabled={status==="sending"}
          style={{width:"100%",padding:16,background:status==="sending"?C.muted:C.gold,color:"#0C0C10",border:"none",borderRadius:16,fontFamily:"'Nunito'",fontWeight:900,fontSize:17,cursor:status==="sending"?"wait":"pointer",boxShadow:"0 6px 20px rgba(245,166,35,.4)",transition:"background .2s"}}
        >
          {status==="sending" ? "⏳ Enviando pedido..." : `🌵 Enviar Pedido · ${fmt(total)}`}
        </button>
      </div>
    </div>
  );

  // ════════════════════════════════════════════════════════════
  //  PANTALLA: MENÚ PRINCIPAL
  // ════════════════════════════════════════════════════════════
  return (
    <div style={{minHeight:"100vh",background:C.bg,color:C.white,fontFamily:"'Nunito',sans-serif",position:"relative",paddingBottom:count>0?"90px":"24px"}}>
      <style>{STYLES}</style>

      {/* ── HEADER ── */}
      <div style={{background:`linear-gradient(to bottom,#111116,${C.bg})`,borderBottom:`1.5px solid ${C.border}`,padding:"18px 16px 14px",position:"sticky",top:0,zIndex:10}}>
        <div style={{maxWidth:600,margin:"0 auto",display:"flex",alignItems:"center",justifyContent:"space-between"}}>
          <div onClick={onLogoTap} style={{userSelect:"none"}}>
            <h1 style={{fontFamily:"'Bebas Neue'",fontSize:34,letterSpacing:3,color:C.gold,lineHeight:1}}>🌵 CACTUS BURGER</h1>
            <p style={{color:C.muted,fontSize:12,marginTop:3}}>Food Truck · Charallave, Edo. Miranda</p>
          </div>
          <div style={{display:"flex",gap:8,alignItems:"center"}}>
            <button onClick={()=>setClubOpen(true)} style={{background:"rgba(245,166,35,.15)",border:`1px solid rgba(245,166,35,.35)`,borderRadius:50,padding:"8px 14px",color:C.gold,fontFamily:"'Nunito'",fontWeight:800,fontSize:13,cursor:"pointer"}}>
              🌵 Club
            </button>
            {count>0 && (
              <button onClick={()=>setCartOpen(true)} style={{background:C.gold,color:"#0C0C10",border:"none",borderRadius:50,padding:"9px 14px",fontFamily:"'Nunito'",fontWeight:900,fontSize:14,cursor:"pointer",boxShadow:"0 4px 16px rgba(245,166,35,.4)"}}>
                🛒 {count} · {fmt(total)}
              </button>
            )}
          </div>
        </div>
      </div>

      {/* ── TABS ── */}
      <div style={{background:C.surface,borderBottom:`1px solid ${C.border}`,position:"sticky",top:67,zIndex:9,overflowX:"auto"}}>
        <div style={{maxWidth:600,margin:"0 auto",display:"flex",padding:"0 8px"}}>
          {MENU_CATEGORIES.map((cat,i)=>(
            <button key={cat.id} className="tab" onClick={()=>setActiveCat(i)} style={{background:"transparent",border:"none",borderBottom:activeCat===i?`2.5px solid ${C.gold}`:"2.5px solid transparent",color:activeCat===i?C.gold:C.light,padding:"11px 11px",fontFamily:"'Nunito'",fontWeight:activeCat===i?800:600,fontSize:12,whiteSpace:"nowrap",cursor:"pointer",display:"flex",alignItems:"center",gap:4}}>
              {cat.emoji} {cat.label}
            </button>
          ))}
        </div>
      </div>

      {/* ── ITEMS ── */}
      <div style={{maxWidth:600,margin:"0 auto",padding:"12px 14px"}}>
        {activeCat===0 && (
          <div style={{background:"linear-gradient(135deg,rgba(245,166,35,.12),rgba(245,166,35,.04))",border:`1px solid rgba(245,166,35,.3)`,borderRadius:14,padding:"10px 14px",display:"flex",alignItems:"center",gap:12,marginBottom:12}}>
            <span style={{fontSize:22}}>🎁</span>
            <div><p style={{fontWeight:800,fontSize:13,color:C.gold}}>¡Combos disponibles!</p><p style={{fontSize:12,color:C.muted}}>Ahorra hasta $2 comprando en combo</p></div>
          </div>
        )}
        <div style={{display:"flex",flexDirection:"column",gap:10}}>
          {cur.items.map((item,i)=>(
            <div key={item.id} className="item-card" style={{background:C.surface,borderRadius:18,border:`1px solid ${C.border}`,padding:15,display:"flex",alignItems:"center",justifyContent:"space-between",gap:12,animation:`fadeInUp .25s ease ${i*.05}s both`}}>
              <div style={{display:"flex",alignItems:"center",gap:14,flex:1,minWidth:0}}>
                <div style={{fontSize:30,flexShrink:0}}>{item.emoji}</div>
                <div style={{minWidth:0}}>
                  <div style={{display:"flex",alignItems:"center",gap:8,flexWrap:"wrap",marginBottom:2}}>
                    <p style={{fontWeight:800,fontSize:14}}>{item.name}</p>
                    {item.isCombo && <span style={{background:C.gold,color:"#0C0C10",fontSize:9,fontWeight:900,padding:"2px 7px",borderRadius:50}}>COMBO</span>}
                  </div>
                  <p style={{color:C.muted,fontSize:12,marginBottom:6}}>{item.desc}</p>
                  <span style={{fontFamily:"'Bebas Neue'",fontSize:22,color:C.gold,letterSpacing:1}}>{fmt(item.price)}</span>
                </div>
              </div>
              <div style={{display:"flex",alignItems:"center",gap:8,flexShrink:0}}>
                {cart[item.id] && <>
                  <button onClick={()=>remItem(item.id)} style={{width:32,height:32,borderRadius:"50%",border:`1.5px solid ${C.border}`,background:"transparent",color:C.light,fontSize:20,cursor:"pointer",display:"flex",alignItems:"center",justifyContent:"center"}}>−</button>
                  <span style={{fontWeight:900,fontSize:16,minWidth:20,textAlign:"center"}}>{cart[item.id].qty}</span>
                </>}
                <button className={`add-btn${popped===item.id?" pop":""}`} onClick={()=>addItem(item)} style={{width:36,height:36,borderRadius:"50%",border:"none",background:C.gold,color:"#0C0C10",fontSize:22,cursor:"pointer",display:"flex",alignItems:"center",justifyContent:"center",fontWeight:900,boxShadow:"0 4px 14px rgba(245,166,35,.35)"}}>+</button>
              </div>
            </div>
          ))}
        </div>
      </div>

      {/* ── FLOATING BAR ── */}
      {count>0 && !cartOpen && (
        <div style={{position:"fixed",bottom:0,left:0,right:0,zIndex:30,padding:"10px 14px 14px",background:`linear-gradient(to top,${C.bg} 60%,transparent)`}}>
          <div style={{maxWidth:600,margin:"0 auto"}}>
            <button onClick={()=>setCartOpen(true)} style={{width:"100%",padding:"14px 20px",background:C.gold,color:"#0C0C10",border:"none",borderRadius:16,fontFamily:"'Nunito'",fontWeight:900,fontSize:16,cursor:"pointer",display:"flex",alignItems:"center",justifyContent:"space-between",boxShadow:"0 6px 24px rgba(245,166,35,.45)"}}>
              <span style={{background:"rgba(0,0,0,.2)",borderRadius:8,padding:"2px 10px",fontSize:14}}>{count} ítem{count!==1?"s":""}</span>
              <span>🛒 Ver Pedido</span>
              <span>{fmt(total)}</span>
            </button>
          </div>
        </div>
      )}

      {/* ── CART DRAWER ── */}
      {cartOpen && (
        <div style={{position:"fixed",inset:0,zIndex:50,display:"flex",flexDirection:"column",justifyContent:"flex-end"}}>
          <div onClick={()=>setCartOpen(false)} style={{position:"absolute",inset:0,background:"rgba(0,0,0,.78)",backdropFilter:"blur(3px)"}}/>
          <div style={{position:"relative",background:C.surface,borderRadius:"24px 24px 0 0",border:`1px solid ${C.border}`,borderBottom:"none",padding:"20px 18px 28px",maxHeight:"75vh",overflowY:"auto",maxWidth:600,width:"100%",margin:"0 auto",animation:"slideUp .28s ease"}}>
            <div style={{width:36,height:4,background:C.border,borderRadius:4,margin:"-8px auto 18px"}}/>
            <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:14}}>
              <h2 style={{fontFamily:"'Bebas Neue'",fontSize:26,letterSpacing:2,color:C.gold}}>Tu Pedido</h2>
              <button onClick={()=>{setCart({});setCartOpen(false);}} style={{background:"transparent",border:`1px solid ${C.border}`,borderRadius:10,color:C.muted,fontSize:12,padding:"6px 14px",cursor:"pointer",fontFamily:"'Nunito'",fontWeight:700}}>Limpiar</button>
            </div>
            <div style={{display:"flex",flexDirection:"column",gap:8,marginBottom:16}}>
              {items.map(item=>(
                <div key={item.id} style={{display:"flex",alignItems:"center",justifyContent:"space-between",padding:"12px 14px",background:C.bg,borderRadius:14}}>
                  <div style={{display:"flex",alignItems:"center",gap:10}}>
                    <span style={{fontSize:20}}>{item.emoji}</span>
                    <div>
                      <p style={{fontWeight:800,fontSize:13}}>{item.name}</p>
                      <p style={{color:C.muted,fontSize:11}}>{fmt(item.price)} c/u</p>
                    </div>
                  </div>
                  <div style={{display:"flex",alignItems:"center",gap:8}}>
                    <button onClick={()=>remItem(item.id)} style={{width:26,height:26,borderRadius:"50%",border:`1px solid ${C.border}`,background:"transparent",color:C.light,fontSize:16,cursor:"pointer",display:"flex",alignItems:"center",justifyContent:"center"}}>−</button>
                    <span style={{fontWeight:900,minWidth:18,textAlign:"center",fontSize:15}}>{item.qty}</span>
                    <button onClick={()=>addItem(item)} style={{width:26,height:26,borderRadius:"50%",border:"none",background:C.gold,color:"#0C0C10",fontSize:16,cursor:"pointer",display:"flex",alignItems:"center",justifyContent:"center",fontWeight:900}}>+</button>
                    <span style={{fontFamily:"'Bebas Neue'",fontSize:17,color:C.gold,minWidth:50,textAlign:"right"}}>{fmt(item.price*item.qty)}</span>
                  </div>
                </div>
              ))}
            </div>
            <div style={{borderTop:`1px solid ${C.border}`,paddingTop:14,marginBottom:16,display:"flex",justifyContent:"space-between",alignItems:"center"}}>
              <span style={{color:C.light,fontSize:15,fontWeight:700}}>Total</span>
              <span style={{fontFamily:"'Bebas Neue'",fontSize:32,color:C.gold,letterSpacing:1}}>{fmt(total)}</span>
            </div>
            <button onClick={()=>{setCartOpen(false);setStatus("checkout");}} style={{width:"100%",padding:15,background:C.gold,color:"#0C0C10",border:"none",borderRadius:16,fontFamily:"'Nunito'",fontWeight:900,fontSize:17,cursor:"pointer",boxShadow:"0 6px 20px rgba(245,166,35,.45)"}}>
              Continuar con el pedido →
            </button>
          </div>
        </div>
      )}

      {/* ── CLUB CACTUS SCREEN ── */}
      {clubOpen && (
        <div style={{position:"fixed",inset:0,zIndex:60,background:C.bg,overflowY:"auto",animation:"fadeInUp .3s ease"}}>
          <div style={{maxWidth:600,margin:"0 auto",paddingBottom:40}}>
            {/* Club header */}
            <div style={{background:`linear-gradient(135deg,rgba(245,166,35,.2),rgba(245,166,35,.05))`,borderBottom:`1px solid rgba(245,166,35,.3)`,padding:"24px 16px 20px"}}>
              <div style={{display:"flex",justifyContent:"space-between",alignItems:"flex-start"}}>
                <div>
                  <h1 style={{fontFamily:"'Bebas Neue'",fontSize:36,color:C.gold,letterSpacing:3,lineHeight:1}}>🌵 CLUB CACTUS</h1>
                  <p style={{color:C.light,fontSize:13,marginTop:4}}>Tu programa de fidelización · 1$ = 1 punto</p>
                </div>
                <button onClick={()=>{setClubOpen(false);setClubResult(null);setClubSearch("");setClubNotFound(false);}} style={{background:"rgba(255,255,255,.1)",border:"none",borderRadius:"50%",width:36,height:36,color:C.light,fontSize:18,cursor:"pointer",display:"flex",alignItems:"center",justifyContent:"center"}}>✕</button>
              </div>
              {/* Buscar */}
              <div style={{display:"flex",gap:8,marginTop:18}}>
                <input
                  value={clubSearch}
                  onChange={e=>setClubSearch(e.target.value)}
                  onKeyDown={e=>e.key==="Enter"&&lookupClub()}
                  placeholder="Ingresa tu número de teléfono"
                  style={{flex:1,background:C.bg,border:`1px solid rgba(245,166,35,.4)`,borderRadius:12,padding:"11px 14px",color:C.white,fontFamily:"'Nunito'",fontSize:14}}
                />
                <button onClick={lookupClub} style={{background:C.gold,color:"#0C0C10",border:"none",borderRadius:12,padding:"11px 18px",fontFamily:"'Nunito'",fontWeight:900,fontSize:14,cursor:"pointer"}}>Buscar</button>
              </div>
            </div>

            <div style={{padding:"16px 14px"}}>
              {/* Recompensas generales */}
              {!clubResult && !clubNotFound && (
                <div>
                  <p style={{color:C.muted,fontSize:13,marginBottom:14,textAlign:"center"}}>Ingresa tu teléfono para ver tus puntos, o descubre las recompensas:</p>
                  <div style={{display:"flex",flexDirection:"column",gap:10}}>
                    {REWARDS.map(r=>(
                      <div key={r.pts} style={{background:C.surface,borderRadius:16,border:`1px solid ${C.border}`,padding:"14px 16px",display:"flex",alignItems:"center",gap:14}}>
                        <div style={{fontSize:28}}>{r.emoji}</div>
                        <div style={{flex:1}}>
                          <p style={{fontWeight:800,fontSize:14}}>{r.label}</p>
                          <p style={{color:C.muted,fontSize:12}}>Al acumular {r.pts} puntos</p>
                        </div>
                        <div style={{background:`${r.color}22`,border:`1px solid ${r.color}55`,borderRadius:50,padding:"6px 12px"}}>
                          <span style={{fontFamily:"'Bebas Neue'",fontSize:18,color:r.color,letterSpacing:1}}>{r.pts} pts</span>
                        </div>
                      </div>
                    ))}
                  </div>
                  <div style={{background:C.surface,borderRadius:14,border:`1px solid ${C.border}`,padding:"14px 16px",marginTop:14,textAlign:"center"}}>
                    <p style={{color:C.light,fontSize:13}}>¿Aún no eres miembro? <strong style={{color:C.gold}}>¡Únete gratis!</strong></p>
                    <p style={{color:C.muted,fontSize:12,marginTop:4}}>Ingresa tu teléfono al confirmar tu próximo pedido y empieza a acumular puntos automáticamente.</p>
                  </div>
                </div>
              )}

              {/* No encontrado */}
              {clubNotFound && (
                <div style={{textAlign:"center",padding:"30px 0"}}>
                  <p style={{fontSize:40,marginBottom:12}}>🤔</p>
                  <p style={{fontWeight:800,fontSize:15,marginBottom:6}}>No encontramos tu número</p>
                  <p style={{color:C.muted,fontSize:13,marginBottom:20}}>¿Primera vez? Ingresa tu teléfono al confirmar tu pedido y te registrarás automáticamente.</p>
                  <button onClick={()=>{setClubOpen(false);}} style={{background:C.gold,color:"#0C0C10",border:"none",borderRadius:14,padding:"12px 24px",fontFamily:"'Nunito'",fontWeight:900,fontSize:14,cursor:"pointer"}}>Hacer un pedido →</button>
                </div>
              )}

              {/* Resultado del miembro */}
              {clubResult && (
                <div style={{display:"flex",flexDirection:"column",gap:14}}>
                  {/* Tarjeta de puntos */}
                  <div style={{background:`linear-gradient(135deg,#1A1410,#211800)`,border:`1.5px solid rgba(245,166,35,.4)`,borderRadius:20,padding:"20px 18px"}}>
                    <div style={{display:"flex",justifyContent:"space-between",alignItems:"flex-start",marginBottom:16}}>
                      <div>
                        <p style={{fontFamily:"'Bebas Neue'",fontSize:14,color:C.muted,letterSpacing:2}}>SOCIO CLUB CACTUS</p>
                        <p style={{fontWeight:900,fontSize:18,marginTop:2}}>{clubResult.name || "Cliente"}</p>
                        <p style={{color:C.muted,fontSize:12}}>{clubResult.phone}</p>
                      </div>
                      <div style={{fontSize:36}}>🌵</div>
                    </div>
                    <p style={{fontFamily:"'Bebas Neue'",fontSize:52,color:C.gold,letterSpacing:2,lineHeight:1}}>{clubResult.points}</p>
                    <p style={{color:C.muted,fontSize:13}}>puntos acumulados</p>
                    {nextReward(clubResult.points) && (
                      <div style={{marginTop:12}}>
                        <div style={{display:"flex",justifyContent:"space-between",fontSize:12,color:C.muted,marginBottom:4}}>
                          <span>Próxima: {nextReward(clubResult.points).label}</span>
                          <span>{clubResult.points}/{nextReward(clubResult.points).pts}</span>
                        </div>
                        <div style={{height:6,background:"rgba(255,255,255,.1)",borderRadius:3}}>
                          <div style={{height:"100%",background:C.gold,borderRadius:3,width:`${Math.min(100,(clubResult.points/nextReward(clubResult.points).pts)*100)}%`,transition:"width .8s ease"}}/>
                        </div>
                      </div>
                    )}
                  </div>

                  {/* Recompensas disponibles */}
                  <div style={{background:C.surface,borderRadius:16,border:`1px solid ${C.border}`,padding:16}}>
                    <h3 style={{fontWeight:800,fontSize:14,color:C.light,marginBottom:12}}>🏆 Tus recompensas</h3>
                    {REWARDS.map(r => {
                      const unlocked = clubResult.points >= r.pts;
                      return (
                        <div key={r.pts} style={{display:"flex",alignItems:"center",gap:12,padding:"10px 0",borderBottom:`1px solid ${C.border}`}}>
                          <span style={{fontSize:24,opacity:unlocked?1:.4}}>{r.emoji}</span>
                          <div style={{flex:1}}>
                            <p style={{fontSize:13,fontWeight:800,color:unlocked?C.white:C.muted}}>{r.label}</p>
                            <p style={{fontSize:11,color:C.muted}}>{r.pts} puntos</p>
                          </div>
                          {unlocked
                            ? <span style={{background:`${C.green}22`,border:`1px solid ${C.green}`,borderRadius:50,padding:"4px 12px",fontSize:12,fontWeight:800,color:C.green}}>¡Disponible!</span>
                            : <span style={{color:C.muted,fontSize:12}}>Faltan {r.pts-clubResult.points} pts</span>
                          }
                        </div>
                      );
                    })}
                  </div>

                  {/* Historial */}
                  {clubResult.history?.length>0 && (
                    <div style={{background:C.surface,borderRadius:16,border:`1px solid ${C.border}`,padding:16}}>
                      <h3 style={{fontWeight:800,fontSize:14,color:C.light,marginBottom:12}}>📋 Historial reciente</h3>
                      {clubResult.history.slice(0,6).map((h,i)=>(
                        <div key={i} style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:8}}>
                          <div>
                            <p style={{fontSize:12,color:C.light}}>{h.orderId==="CANJE"?"Canje de recompensa":`Pedido #${pad(h.orderId)}`}</p>
                            <p style={{fontSize:11,color:C.muted}}>{h.date}</p>
                          </div>
                          <span style={{fontFamily:"'Bebas Neue'",fontSize:16,color:h.pts>0?C.green:C.red,letterSpacing:.5}}>
                            {h.pts>0?"+":""}{h.pts} pts
                          </span>
                        </div>
                      ))}
                    </div>
                  )}
                </div>
              )}
            </div>
          </div>
        </div>
      )}

      {/* ── PIN MODAL ── */}
      {showPin && (
        <div style={{position:"fixed",inset:0,zIndex:70,display:"flex",alignItems:"center",justifyContent:"center",background:"rgba(0,0,0,.88)",backdropFilter:"blur(5px)"}}>
          <div style={{background:C.surface,borderRadius:24,padding:"28px 24px",width:300,border:`1px solid ${C.border}`,textAlign:"center",animation:"fadeInUp .3s ease"}}>
            <p style={{fontFamily:"'Bebas Neue'",fontSize:22,color:C.gold,letterSpacing:2,marginBottom:4}}>🔒 PANEL DUEÑO</p>
            <p style={{color:C.muted,fontSize:12,marginBottom:22}}>Ingresa tu PIN de acceso</p>
            <div className={pinErr?"shake":""} style={{display:"flex",justifyContent:"center",gap:12,marginBottom:24}}>
              {[0,1,2,3].map(i=><div key={i} style={{width:13,height:13,borderRadius:"50%",background:pinInput.length>i?C.gold:C.border,transition:"background .15s"}}/>)}
            </div>
            <div style={{display:"grid",gridTemplateColumns:"repeat(3,1fr)",gap:10,marginBottom:16}}>
              {["1","2","3","4","5","6","7","8","9","","0","⌫"].map((k,i)=>(
                <button key={i} className="pkey" onClick={()=>k&&tapKey(k)} style={{height:52,background:k?"":  "transparent",border:k?`1px solid ${C.border}`:"none",borderRadius:14,color:C.white,fontSize:k==="⌫"?18:22,fontFamily:"'Nunito'",fontWeight:700,cursor:k?"pointer":"default",display:"flex",alignItems:"center",justifyContent:"center"}}>{k}</button>
              ))}
            </div>
            <button onClick={()=>{setShowPin(false);setPinInput("");}} style={{background:"transparent",border:"none",color:C.muted,fontSize:13,cursor:"pointer",fontFamily:"'Nunito'"}}>Cancelar</button>
          </div>
        </div>
      )}

      {/* ════════════════════════════════════════════════════
           PANEL DUEÑO
         ════════════════════════════════════════════════════ */}
      {ownerOpen && (
        <div style={{position:"fixed",inset:0,zIndex:65,background:C.bg,overflowY:"auto",animation:"fadeInUp .3s ease"}}>
          <div style={{maxWidth:600,margin:"0 auto",paddingBottom:40}}>
            {/* Header */}
            <div style={{background:C.surface,borderBottom:`1.5px solid ${C.border}`,padding:"16px 16px 0",position:"sticky",top:0,zIndex:5}}>
              <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:12}}>
                <div>
                  <h2 style={{fontFamily:"'Bebas Neue'",fontSize:24,color:C.gold,letterSpacing:2}}>🌵 Panel de Ventas</h2>
                  <p style={{color:C.muted,fontSize:12}}>{new Date().toLocaleDateString("es-VE",{weekday:"long",day:"numeric",month:"long"})}</p>
                </div>
                <button onClick={()=>setOwnerOpen(false)} style={{background:C.hi,border:`1px solid ${C.border}`,borderRadius:"50%",width:36,height:36,color:C.light,fontSize:18,cursor:"pointer",display:"flex",alignItems:"center",justifyContent:"center"}}>✕</button>
              </div>
              <div style={{display:"flex"}}>
                {[["today","📊 Hoy"],["history","📋 Historial"],["club","🌵 Club"],["config","⚙️ Config"]].map(([id,lbl])=>(
                  <button key={id} onClick={()=>setOwnerTab(id)} style={{flex:1,background:"transparent",border:"none",borderBottom:ownerTab===id?`2.5px solid ${C.gold}`:"2.5px solid transparent",color:ownerTab===id?C.gold:C.muted,padding:"10px 0",fontFamily:"'Nunito'",fontWeight:ownerTab===id?800:600,fontSize:12,cursor:"pointer"}}>
                    {lbl}
                  </button>
                ))}
              </div>
            </div>

            <div style={{padding:"16px 14px"}}>

              {/* ── HOY ── */}
              {ownerTab==="today" && (
                <div style={{display:"flex",flexDirection:"column",gap:14}}>
                  <div style={{display:"grid",gridTemplateColumns:"1fr 1fr 1fr",gap:10}}>
                    {[
                      {label:"Pedidos Hoy", val:todayOrders.length,  emoji:"📋", color:C.gold},
                      {label:"Ingresos Hoy",val:fmt(todayRevenue),   emoji:"💰", color:C.green},
                      {label:"Total Gral.", val:fmt(totalRevenue),   emoji:"🏆", color:C.blue},
                    ].map(s=>(
                      <div key={s.label} style={{background:C.surface,borderRadius:16,padding:"14px 10px",border:`1px solid ${C.border}`,textAlign:"center"}}>
                        <div style={{fontSize:20,marginBottom:6}}>{s.emoji}</div>
                        <p style={{fontFamily:"'Bebas Neue'",fontSize:19,color:s.color}}>{s.val}</p>
                        <p style={{color:C.muted,fontSize:10,marginTop:2}}>{s.label}</p>
                      </div>
                    ))}
                  </div>
                  {topItems.length>0 && (
                    <div style={{background:C.surface,borderRadius:16,border:`1px solid ${C.border}`,padding:16}}>
                      <h3 style={{fontWeight:800,fontSize:14,color:C.light,marginBottom:12}}>🔥 Más vendidos</h3>
                      {topItems.map(([name,qty],i)=>(
                        <div key={name} style={{display:"flex",alignItems:"center",gap:10,marginBottom:10}}>
                          <span style={{fontFamily:"'Bebas Neue'",fontSize:18,color:i===0?C.gold:C.muted,minWidth:24}}>#{i+1}</span>
                          <div style={{flex:1}}>
                            <div style={{display:"flex",justifyContent:"space-between",marginBottom:4}}>
                              <span style={{fontSize:13,fontWeight:700}}>{name}</span>
                              <span style={{fontSize:13,color:C.gold,fontWeight:800}}>{qty} uds.</span>
                            </div>
                            <div style={{height:4,background:C.border,borderRadius:4}}>
                              <div style={{height:"100%",background:i===0?C.gold:C.muted,borderRadius:4,width:`${(qty/topItems[0][1])*100}%`,transition:"width .6s ease"}}/>
                            </div>
                          </div>
                        </div>
                      ))}
                    </div>
                  )}
                  <div style={{background:C.surface,borderRadius:16,border:`1px solid ${C.border}`,padding:16}}>
                    <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:12}}>
                      <h3 style={{fontWeight:800,fontSize:14,color:C.light}}>📋 Pedidos de Hoy ({todayOrders.length})</h3>
                      <button onClick={loadOrders} style={{background:"transparent",border:"none",color:C.muted,fontSize:12,cursor:"pointer",fontFamily:"'Nunito'"}}>↻</button>
                    </div>
                    {todayOrders.length===0 ? <p style={{color:C.muted,fontSize:13,textAlign:"center",padding:"20px 0"}}>Sin pedidos hoy todavía 🌵</p>
                    : [...todayOrders].reverse().map(o=>(
                      <div key={o.id} onClick={()=>setSelOrder(selOrder?.id===o.id?null:o)} style={{background:C.bg,borderRadius:12,padding:12,marginBottom:8,cursor:"pointer",border:`1px solid ${selOrder?.id===o.id?C.gold:C.border}`,transition:"border-color .2s"}}>
                        <div style={{display:"flex",justifyContent:"space-between",alignItems:"center"}}>
                          <div>
                            <p style={{fontWeight:800,fontSize:13}}>#{pad(o.id)} · {o.customer?.name||"—"}</p>
                            <p style={{color:C.muted,fontSize:11}}>{o.customer?.delivery==="delivery"?"🛵 Delivery":"🏪 Retiro"} · {PAYMENTS.find(p=>p.id===o.customer?.payment)?.label||o.customer?.payment}</p>
                          </div>
                          <span style={{fontFamily:"'Bebas Neue'",fontSize:18,color:C.gold}}>{fmt(o.total)}</span>
                        </div>
                        {selOrder?.id===o.id && (
                          <div style={{marginTop:10,paddingTop:10,borderTop:`1px solid ${C.border}`}}>
                            {o.items?.map(i=><p key={i.id+i.name} style={{fontSize:12,color:C.light,marginBottom:3}}>{i.qty}x {i.name}</p>)}
                            {o.customer?.address && <p style={{fontSize:12,color:C.blue,marginTop:4}}>📍 {o.customer.address}</p>}
                            {o.notes && <p style={{fontSize:12,color:C.gold,marginTop:4}}>📝 {o.notes}</p>}
                            {o.customer?.clubPhone && <p style={{fontSize:12,color:C.gold,marginTop:4}}>🌵 Club: {o.customer.clubPhone}</p>}
                          </div>
                        )}
                      </div>
                    ))}
                  </div>
                  <button onClick={clearOrders} style={{background:"transparent",border:`1px solid ${C.red}`,borderRadius:12,color:C.red,fontFamily:"'Nunito'",fontWeight:700,fontSize:13,padding:10,cursor:"pointer"}}>🗑️ Limpiar historial</button>
                </div>
              )}

              {/* ── HISTORIAL ── */}
              {ownerTab==="history" && (
                <div>
                  <p style={{color:C.muted,fontSize:13,marginBottom:12}}>{orders.length} pedido{orders.length!==1?"s":""} en total</p>
                  {[...orders].reverse().map(o=>(
                    <div key={o.id} onClick={()=>setSelOrder(selOrder?.id===o.id?null:o)} style={{background:C.surface,borderRadius:14,padding:14,marginBottom:10,cursor:"pointer",border:`1px solid ${selOrder?.id===o.id?C.gold:C.border}`,transition:"border-color .2s"}}>
                      <div style={{display:"flex",justifyContent:"space-between",alignItems:"flex-start"}}>
                        <div>
                          <p style={{fontWeight:800,fontSize:13}}>#{pad(o.id)} · {o.customer?.name||"—"}</p>
                          <p style={{color:C.muted,fontSize:11}}>{new Date(o.timestamp).toLocaleString("es-VE",{day:"2-digit",month:"short",hour:"2-digit",minute:"2-digit"})}</p>
                        </div>
                        <span style={{fontFamily:"'Bebas Neue'",fontSize:20,color:C.gold}}>{fmt(o.total)}</span>
                      </div>
                      {selOrder?.id===o.id && (
                        <div style={{marginTop:10,paddingTop:10,borderTop:`1px solid ${C.border}`}}>
                          {o.items?.map(i=><p key={i.id+i.name} style={{fontSize:12,color:C.light,marginBottom:3}}>{i.qty}x {i.name}</p>)}
                          {o.customer?.address && <p style={{fontSize:12,color:C.blue,marginTop:4}}>📍 {o.customer.address}</p>}
                          {o.notes && <p style={{fontSize:12,color:C.gold,marginTop:4}}>📝 {o.notes}</p>}
                        </div>
                      )}
                    </div>
                  ))}
                </div>
              )}

              {/* ── CLUB ── */}
              {ownerTab==="club" && (
                <div style={{display:"flex",flexDirection:"column",gap:14}}>
                  <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10}}>
                    <div style={{background:C.surface,borderRadius:16,padding:"14px",border:`1px solid ${C.border}`,textAlign:"center"}}>
                      <p style={{fontSize:24,marginBottom:4}}>🌵</p>
                      <p style={{fontFamily:"'Bebas Neue'",fontSize:26,color:C.gold}}>{members.length}</p>
                      <p style={{color:C.muted,fontSize:11}}>Miembros</p>
                    </div>
                    <div style={{background:C.surface,borderRadius:16,padding:"14px",border:`1px solid ${C.border}`,textAlign:"center"}}>
                      <p style={{fontSize:24,marginBottom:4}}>⭐</p>
                      <p style={{fontFamily:"'Bebas Neue'",fontSize:26,color:C.purple}}>{members.reduce((s,m)=>s+m.points,0)}</p>
                      <p style={{color:C.muted,fontSize:11}}>Pts Totales</p>
                    </div>
                  </div>
                  <button onClick={loadMembers} style={{background:"transparent",border:`1px solid ${C.border}`,borderRadius:12,color:C.muted,fontFamily:"'Nunito'",fontWeight:700,fontSize:13,padding:10,cursor:"pointer"}}>↻ Actualizar lista</button>
                  {members.length===0 ? <p style={{color:C.muted,fontSize:13,textAlign:"center",padding:"30px 0"}}>Sin miembros registrados aún.</p>
                  : members.sort((a,b)=>b.points-a.points).map(m=>(
                    <div key={m.phone} style={{background:C.surface,borderRadius:16,border:`1px solid ${selMember?.phone===m.phone?C.gold:C.border}`,padding:16,cursor:"pointer",transition:"border-color .2s"}} onClick={()=>setSelMember(selMember?.phone===m.phone?null:m)}>
                      <div style={{display:"flex",justifyContent:"space-between",alignItems:"center"}}>
                        <div>
                          <p style={{fontWeight:800,fontSize:14}}>{m.name||"Sin nombre"}</p>
                          <p style={{color:C.muted,fontSize:12}}>{m.phone}</p>
                        </div>
                        <div style={{textAlign:"right"}}>
                          <p style={{fontFamily:"'Bebas Neue'",fontSize:22,color:C.gold,letterSpacing:1}}>{m.points} pts</p>
                          {REWARDS.filter(r=>r.pts<=m.points).length>0 && (
                            <p style={{color:C.green,fontSize:11,fontWeight:700}}>🏆 {REWARDS.filter(r=>r.pts<=m.points).length} premio{REWARDS.filter(r=>r.pts<=m.points).length!==1?"s":""} disponible{REWARDS.filter(r=>r.pts<=m.points).length!==1?"s":""}</p>
                          )}
                        </div>
                      </div>
                      {selMember?.phone===m.phone && (
                        <div style={{marginTop:14,paddingTop:14,borderTop:`1px solid ${C.border}`}}>
                          <p style={{fontSize:13,fontWeight:700,color:C.light,marginBottom:10}}>Canjear puntos (descuento recompensa):</p>
                          <div style={{display:"flex",gap:8}}>
                            <input
                              value={deductAmt}
                              onChange={e=>setDeductAmt(e.target.value.replace(/\D/g,""))}
                              placeholder="Pts a descontar"
                              onClick={e=>e.stopPropagation()}
                              style={{flex:1,background:C.bg,border:`1px solid ${C.border}`,borderRadius:10,padding:"9px 12px",color:C.white,fontFamily:"'Nunito'",fontSize:13}}
                            />
                            <button onClick={e=>{e.stopPropagation();handleDeduct();}} style={{background:deductOk?C.green:C.red,color:"#fff",border:"none",borderRadius:10,padding:"9px 16px",fontFamily:"'Nunito'",fontWeight:800,fontSize:13,cursor:"pointer",transition:"background .2s"}}>
                              {deductOk?"✅":"Canjear"}
                            </button>
                          </div>
                          {REWARDS.filter(r=>r.pts<=m.points).map(r=>(
                            <button key={r.pts} onClick={e=>{e.stopPropagation();setDeductAmt(String(r.pts));}} style={{marginTop:8,marginRight:8,background:`${r.color}22`,border:`1px solid ${r.color}55`,borderRadius:50,padding:"5px 12px",color:r.color,fontFamily:"'Nunito'",fontWeight:700,fontSize:12,cursor:"pointer"}}>
                              {r.emoji} {r.label} (-{r.pts}pts)
                            </button>
                          ))}
                        </div>
                      )}
                    </div>
                  ))}
                </div>
              )}

              {/* ── CONFIG ── */}
              {ownerTab==="config" && (
                <div style={{display:"flex",flexDirection:"column",gap:14}}>
                  <div style={{background:C.surface,borderRadius:16,border:`1px solid ${C.border}`,padding:18}}>
                    <h3 style={{fontWeight:800,fontSize:14,color:C.gold,marginBottom:16}}>⚙️ Configuración</h3>
                    {[
                      {label:"📲 WhatsApp",sub:`Actual: ${cfgWA}`,val:newWA,set:setNewWA,ph:"58412XXXXXXX"},
                      {label:"🔔 Tema ntfy.sh",sub:`Actual: ${cfgNtfy}`,val:newNtfy,set:setNewNtfy,ph:"mi-tema"},
                    ].map(f=>(
                      <div key={f.label} style={{marginBottom:14}}>
                        <label style={{fontSize:13,fontWeight:700,color:C.light,display:"block",marginBottom:3}}>{f.label}</label>
                        <p style={{color:C.muted,fontSize:11,marginBottom:6}}>{f.sub}</p>
                        <input value={f.val} onChange={e=>f.set(e.target.value)} placeholder={f.ph} style={{width:"100%",background:C.bg,border:`1px solid ${C.border}`,borderRadius:10,padding:"10px 12px",color:C.white,fontFamily:"'Nunito'",fontSize:13}}/>
                      </div>
                    ))}
                    <div style={{marginBottom:18}}>
                      <label style={{fontSize:13,fontWeight:700,color:C.light,display:"block",marginBottom:3}}>🔐 Nuevo PIN (4 dígitos)</label>
                      <input value={newPin} onChange={e=>setNewPin(e.target.value.replace(/\D/g,"").slice(0,4))} placeholder="Nuevo PIN" style={{width:"100%",background:C.bg,border:`1px solid ${C.border}`,borderRadius:10,padding:"10px 12px",color:C.white,fontFamily:"'Nunito'",fontSize:16,letterSpacing:8}}/>
                    </div>
                    <button onClick={saveConfig} style={{width:"100%",padding:13,background:cfgSaved?C.green:C.gold,color:"#0C0C10",border:"none",borderRadius:14,fontFamily:"'Nunito'",fontWeight:900,fontSize:15,cursor:"pointer",transition:"background .3s"}}>
                      {cfgSaved?"✅ ¡Guardado!":"Guardar Cambios"}
                    </button>
                  </div>
                </div>
              )}

            </div>
          </div>
        </div>
      )}
    </div>
  );
}
