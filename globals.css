"use client";

import React, { useState, useEffect, useRef, useCallback } from "react";
import {
  Home, Briefcase, Car, TrendingUp, Users, Bell, Settings, Send,
  Sparkles, Check, Search, ChevronRight, X, Loader2, Circle,
  CalendarDays, Menu, ArrowUpRight, ArrowDownRight, ChevronLeft, AlertTriangle
} from "lucide-react";
import {
  BarChart, Bar, PieChart, Pie, Cell, XAxis, YAxis, CartesianGrid,
  Tooltip, ResponsiveContainer
} from "recharts";
import { mockInterpret } from "../lib/mockNlu";

/* ------------------------------------------------------------------ */
/*  tokens                                                              */
/* ------------------------------------------------------------------ */

const NAVY = "#0B1F3D";
const NAVY_SOFT = "#16305A";
const SKY = "#4C8DFF";
const OK = "#2FB380";
const WARN = "#E3A008";
const DANGER = "#E36A6A";

function useGoogleFonts() {
  useEffect(() => {
    if (document.getElementById("horai-fonts")) return;
    const link = document.createElement("link");
    link.id = "horai-fonts";
    link.rel = "stylesheet";
    link.href =
      "https://fonts.googleapis.com/css2?family=Sora:wght@400;600;700;800&family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500;600&display=swap";
    document.head.appendChild(link);
  }, []);
}

/* ------------------------------------------------------------------ */
/*  date helpers (Japan time)                                           */
/* ------------------------------------------------------------------ */

function getJapanDateString() {
  return new Intl.DateTimeFormat("sv-SE", {
    timeZone: "Asia/Tokyo",
    year: "numeric",
    month: "2-digit",
    day: "2-digit",
  }).format(new Date());
}

function addDays(dateStr, days) {
  const [y, m, d] = dateStr.split("-").map(Number);
  const dt = new Date(Date.UTC(y, m - 1, d));
  dt.setUTCDate(dt.getUTCDate() + days);
  return dt.toISOString().slice(0, 10);
}

// "2026-07-14" -> "7/14"
function md(dateStr) {
  if (!dateStr) return "-";
  const [, m, d] = dateStr.split("-").map(Number);
  return `${m}/${d}`;
}

// slash range for confirmed bookings, e.g. "7/14〜7/18" (or "7/14" for a single day)
function formatSlashRange(start, end) {
  if (!start || !end) return "未指定";
  const s = md(start);
  const e = md(end);
  return s === e ? s : `${s}〜${e}`;
}

// natural-language range for conflict messages, e.g. "7月14日から18日まで"
function formatJPRange(start, end) {
  const [, ms, ds] = start.split("-").map(Number);
  const [, me, de] = end.split("-").map(Number);
  if (start === end) return `${ms}月${ds}日`;
  if (ms === me) return `${ms}月${ds}日から${de}日まで`;
  return `${ms}月${ds}日から${me}月${de}日まで`;
}

function rangesOverlap(aStart, aEnd, bStart, bEnd) {
  return aStart <= bEnd && aEnd >= bStart;
}

/* ------------------------------------------------------------------ */
/*  seed data                                                           */
/* ------------------------------------------------------------------ */

const SALES_REPS = ["西出", "小町", "久保", "村井", "田中", "アオキ"];
const MANAGERS = ["池田", "吉田", "前川", "北村", "中田", "若林"];
const CURRENT_USER = "西出"; // demo persona — the person typing into the assistant
const ALLOWED_STATUSES = ["見積中", "商談中", "受注", "契約更新"];

const SEED_TODAY = getJapanDateString();

const SEED_CASES = [
  { id: "c1", name: "テクノポート", owner: "小町", status: "受注", sales: 4200000, profit: 1050000, deadline: addDays(SEED_TODAY, 4) },
  { id: "c2", name: "アイリス", owner: "久保", status: "商談中", sales: 1800000, profit: 420000, deadline: addDays(SEED_TODAY, 20) },
  { id: "c3", name: "イチボ", owner: "村井", status: "見積中", sales: 950000, profit: 190000, deadline: addDays(SEED_TODAY, 16) },
  { id: "c4", name: "テルメ", owner: "田中", status: "受注", sales: 3300000, profit: 810000, deadline: addDays(SEED_TODAY, 28) },
  { id: "c5", name: "ダイワ通信", owner: "西出", status: "契約更新", sales: 5600000, profit: 1340000, deadline: addDays(SEED_TODAY, 45) },
  { id: "c6", name: "○○ホテル", owner: "小町", status: "見積中", sales: 1200000, profit: 260000, deadline: addDays(SEED_TODAY, 5) },
  { id: "c7", name: "△△薬局", owner: "久保", status: "見積中", sales: 680000, profit: 140000, deadline: addDays(SEED_TODAY, 6) },
];

const SEED_VEHICLES = [
  { id: "v1", name: "プリウス", plate: "品川 300 あ 12-34", bookings: [{ id: "b1", startDate: addDays(SEED_TODAY, 3), endDate: addDays(SEED_TODAY, 4), who: "田中" }] },
  { id: "v2", name: "ハイエース", plate: "品川 400 い 56-78", bookings: [{ id: "b2", startDate: addDays(SEED_TODAY, 8), endDate: addDays(SEED_TODAY, 10), who: "久保" }] },
  { id: "v3", name: "フリード", plate: "品川 500 う 90-12", bookings: [] },
  { id: "v4", name: "N-BOX", plate: "品川 100 え 34-56", bookings: [] },
];

const EMPLOYEES = [
  ...SALES_REPS.map((n, i) => ({ id: `e-s-${i}`, name: n, dept: "営業部", role: "営業担当", cases: 3 + (i % 4) })),
  ...MANAGERS.map((n, i) => ({ id: `e-m-${i}`, name: n, dept: "管理部", role: "監理担当", cases: 2 + (i % 3) })),
];

const SALES_BY_REP = SALES_REPS.map((name, i) => ({
  name,
  売上: [3200000, 4600000, 2100000, 3900000, 5100000, 2800000][i],
  粗利: [780000, 1120000, 480000, 940000, 1260000, 690000][i],
}));

const PIE_COLORS = [SKY, NAVY_SOFT, OK, WARN, "#8B6FE8"];

const TABS = [
  { key: "home", label: "ホーム", icon: Home },
  { key: "cases", label: "案件", icon: Briefcase },
  { key: "vehicles", label: "車両", icon: Car },
  { key: "sales", label: "売上", icon: TrendingUp },
  { key: "more", label: "もっと", icon: Menu },
];

const MORE_ITEMS = [
  { key: "employees", label: "社員一覧", icon: Users },
  { key: "notifications", label: "通知", icon: Bell },
  { key: "settings", label: "設定", icon: Settings },
];

const STATUS_COLOR = { 受注: OK, 商談中: SKY, 見積中: WARN, 契約更新: "#8B6FE8" };

const yen = (n) => `¥${Number(n || 0).toLocaleString("ja-JP")}`;

function CountUp({ to, format }) {
  const [val, setVal] = useState(0);
  useEffect(() => {
    let raf;
    const start = performance.now();
    const dur = 800;
    const tick = (now) => {
      const p = Math.min(1, (now - start) / dur);
      setVal(Math.round(to * (1 - Math.pow(1 - p, 3))));
      if (p < 1) raf = requestAnimationFrame(tick);
    };
    raf = requestAnimationFrame(tick);
    return () => cancelAnimationFrame(raf);
  }, [to]);
  return <span>{format ? format(val) : val.toLocaleString("ja-JP")}</span>;
}

const STORAGE_KEY = "horai-app-state-v3";

/* ------------------------------------------------------------------ */
/*  server call — /api/ai (Anthropic key never touches the browser)     */
/* ------------------------------------------------------------------ */

async function interpretRequest(userText, { cases, vehicles }) {
  const res = await fetch("/api/ai", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      userText,
      today: getJapanDateString(),
      currentUser: CURRENT_USER,
      salesReps: SALES_REPS,
      managers: MANAGERS,
      // 最小限のデータのみ送信する（案件は名前と期限のみ、車両は名前のみ）
      cases: cases.map((c) => ({ name: c.name, deadline: c.deadline })),
      vehicles: vehicles.map((v) => ({ name: v.name })),
    }),
  });

  let data = null;
  try {
    data = await res.json();
  } catch (e) {
    data = null;
  }

  if (!res.ok) {
    const message = (data && data.error) || "AIとの通信中にエラーが発生しました。";
    throw new Error(message);
  }
  return data;
}

/* ------------------------------------------------------------------ */
/*  root                                                                 */
/* ------------------------------------------------------------------ */

export default function Page() {
  useGoogleFonts();

  const [screen, setScreen] = useState("home");
  const [moreOpen, setMoreOpen] = useState(false);
  const [subScreen, setSubScreen] = useState(null);

  const [cases, setCases] = useState(SEED_CASES);
  const [vehicles, setVehicles] = useState(SEED_VEHICLES);
  const [notifications, setNotifications] = useState([]);
  const [unread, setUnread] = useState(0);
  const [flashCaseId, setFlashCaseId] = useState(null);
  const [flashBookingId, setFlashBookingId] = useState(null);
  const [salesReady, setSalesReady] = useState(true);
  const [hydrated, setHydrated] = useState(false);

  const [chatLog, setChatLog] = useState([
    { id: "sys0", type: "text", role: "ai", text: "こんにちは。業務内容を自然な文章で入力してください。車両予約、案件登録、見積期限の確認、通知依頼、売上の集計に対応します。" },
  ]);
  const [input, setInput] = useState("");
  const [isBusy, setIsBusy] = useState(false);
  const [mockMode, setMockMode] = useState(null); // null = 未確認, true/false = 判明済み
  const [presentationMode, setPresentationMode] = useState(false); // true = 外部通信なしで確実に動作させる
  const [googleStatus, setGoogleStatus] = useState({ calendarConfigured: false, sheetsConfigured: false });
  const [autoExportSheets, setAutoExportSheets] = useState(false);
  const [sheetsExport, setSheetsExport] = useState({ status: "idle", message: "", time: null }); // idle | loading | success | error

  const logEndRef = useRef(null);
  useEffect(() => {
    logEndRef.current?.scrollIntoView({ behavior: "smooth", block: "end" });
  }, [chatLog]);

  /* ---- check which Google integrations are configured on the server ---- */
  useEffect(() => {
    (async () => {
      try {
        const res = await fetch("/api/google/status");
        if (res.ok) {
          const data = await res.json();
          setGoogleStatus({
            calendarConfigured: !!data.calendarConfigured,
            sheetsConfigured: !!data.sheetsConfigured,
          });
        }
      } catch (e) {
        // 未設定として扱う（アプリの動作には影響しない）
      }
    })();
  }, []);

  /* ---- load persisted state once on mount (localStorage, browser-only) ---- */
  useEffect(() => {
    try {
      const raw = window.localStorage.getItem(STORAGE_KEY);
      if (raw) {
        const saved = JSON.parse(raw);
        if (saved && typeof saved === "object") {
          if (Array.isArray(saved.cases)) setCases(saved.cases);
          if (Array.isArray(saved.vehicles)) setVehicles(saved.vehicles);
          if (Array.isArray(saved.notifications)) setNotifications(saved.notifications);
          if (typeof saved.unread === "number") setUnread(saved.unread);
          if (Array.isArray(saved.chatLog) && saved.chatLog.length) setChatLog(saved.chatLog);
          if (typeof saved.presentationMode === "boolean") setPresentationMode(saved.presentationMode);
          if (typeof saved.autoExportSheets === "boolean") setAutoExportSheets(saved.autoExportSheets);
        }
      }
    } catch (e) {
      // 壊れた保存データやlocalStorage無効環境でも、アプリはシード状態のまま動作を続ける
      console.error("HORAI AI: failed to load saved state ->", e);
    } finally {
      setHydrated(true);
    }
  }, []);

  /* ---- persist state (debounced) whenever it changes ---- */
  useEffect(() => {
    if (!hydrated) return;
    const t = setTimeout(() => {
      try {
        window.localStorage.setItem(
          STORAGE_KEY,
          JSON.stringify({ cases, vehicles, notifications, unread, chatLog, presentationMode, autoExportSheets })
        );
      } catch (e) {
        // 保存できなくてもアプリの動作は継続する
        console.error("HORAI AI: failed to save state ->", e);
      }
    }, 500);
    return () => clearTimeout(t);
  }, [cases, vehicles, notifications, unread, chatLog, presentationMode, autoExportSheets, hydrated]);

  /* ---- opening the notifications screen clears the unread badge ---- */
  useEffect(() => {
    if (subScreen === "notifications") setUnread(0);
  }, [subScreen]);

  const pushUser = (text) => setChatLog((l) => [...l, { id: `u-${Date.now()}`, type: "text", role: "user", text }]);
  const pushAI = (text) => setChatLog((l) => [...l, { id: `ai-${Date.now()}`, type: "text", role: "ai", text }]);

  const syncBookingToCalendar = async (vehicleName, startDate, endDate, who) => {
    const res = await fetch("/api/calendar/create-event", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ vehicleName, startDate, endDate, who }),
    });
    if (!res.ok) {
      let msg = "Googleカレンダーへの登録に失敗しました。";
      try {
        const data = await res.json();
        if (data && data.error) msg = data.error;
      } catch (e) {}
      throw new Error(msg);
    }
    return res.json();
  };

  const exportToSheets = useCallback(async () => {
    setSheetsExport({ status: "loading", message: "", time: null });
    try {
      const res = await fetch("/api/sheets/export", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ cases, vehicles, notifications }),
      });
      if (!res.ok) {
        let msg = "スプレッドシートへの書き出しに失敗しました。";
        try {
          const data = await res.json();
          if (data && data.error) msg = data.error;
        } catch (e) {}
        throw new Error(msg);
      }
      setSheetsExport({ status: "success", message: "", time: new Date() });
    } catch (e) {
      console.error("HORAI AI: sheets export failed ->", e);
      setSheetsExport({ status: "error", message: e.message || "書き出しに失敗しました。", time: new Date() });
    }
  }, [cases, vehicles, notifications]);

  /* ---- optional: automatically re-export to Sheets when data changes ---- */
  useEffect(() => {
    if (!hydrated || !autoExportSheets || !googleStatus.sheetsConfigured) return;
    const t = setTimeout(() => {
      exportToSheets();
    }, 1500);
    return () => clearTimeout(t);
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [cases, vehicles, notifications, autoExportSheets, googleStatus.sheetsConfigured, hydrated]);

  const setProcStatus = (procId, idx, status) => {
    setChatLog((l) =>
      l.map((m) => {
        if (m.id !== procId) return m;
        const steps = m.steps.map((s, i) => (i === idx ? { ...s, status } : s));
        return { ...m, steps };
      })
    );
  };

  const wait = (ms) => new Promise((r) => setTimeout(r, ms));

  const addStep = async (procId, label) => {
    setChatLog((l) => l.map((m) => (m.id === procId ? { ...m, steps: [...m.steps, { label, status: "active" }] } : m)));
    await wait(450);
    setChatLog((l) =>
      l.map((m) =>
        m.id === procId
          ? { ...m, steps: m.steps.map((s, i) => (i === m.steps.length - 1 ? { ...s, status: "done" } : s)) }
          : m
      )
    );
  };

  const handleSubmit = async () => {
    const text = input.trim();
    if (!text || isBusy) return;
    pushUser(text);
    setInput("");
    setIsBusy(true);

    const procId = `proc-${Date.now()}`;
    setChatLog((l) => [
      ...l,
      { id: procId, type: "process", steps: [{ label: "AIが内容を解析しています", status: "active" }] },
    ]);

    let parsed;
    if (presentationMode) {
      // プレゼンモード：外部通信を一切行わず、ブラウザ内で確定的に解析する。
      // ネットワークやAPIの状態に関わらず必ず応答する。
      await wait(500); // 通常のAI応答と同じ体感速度に揃える演出
      parsed = mockInterpret(
        text,
        { today: getJapanDateString(), salesReps: SALES_REPS, managers: MANAGERS, cases, vehicles },
        { silent: true }
      );
    } else {
      try {
        parsed = await interpretRequest(text, { cases, vehicles });
      } catch (e) {
        console.error("HORAI AI: interpretRequest failed ->", e);
        setProcStatus(procId, 0, "error");
        setIsBusy(false);
        pushAI(e.message || "AIとの通信中にエラーが発生しました。");
        return;
      }
    }

    setProcStatus(procId, 0, "done");
    if (typeof parsed._mock === "boolean") setMockMode(parsed._mock);

    try {
      switch (parsed.intent) {
        case "book_vehicle": {
          const match = vehicles.find((v) => parsed.vehicle_name && v.name.includes(parsed.vehicle_name));
          if (!match) {
            pushAI(`該当する車両が見つかりませんでした。登録車両：${vehicles.map((v) => v.name).join("、")}`);
            break;
          }
          if (!parsed.start_date || !parsed.end_date) {
            pushAI("予約したい日付を教えてください。");
            break;
          }

          await addStep(procId, "空き状況を確認");
          const conflict = match.bookings.find((b) => rangesOverlap(parsed.start_date, parsed.end_date, b.startDate, b.endDate));

          if (conflict) {
            const alternatives = vehicles.filter(
              (v) => v.id !== match.id && !v.bookings.some((b) => rangesOverlap(parsed.start_date, parsed.end_date, b.startDate, b.endDate))
            );
            let msg = `${match.name}は${formatJPRange(parsed.start_date, parsed.end_date)}予約済みです。`;
            msg += alternatives.length
              ? `同じ期間は${alternatives.map((a) => a.name).join("と")}が空いています。`
              : `同じ期間に空いている車両を確認してください。`;
            pushAI(msg);
            break;
          }

          await addStep(procId, "予約を登録");
          const bid = `b-${Date.now()}`;
          const bookingOwner = parsed.owner || CURRENT_USER;
          setVehicles((vs) =>
            vs.map((v) =>
              v.id === match.id
                ? { ...v, bookings: [...v.bookings, { id: bid, startDate: parsed.start_date, endDate: parsed.end_date, who: bookingOwner }] }
                : v
            )
          );
          setFlashBookingId(bid);

          let calendarNote = "";
          if (googleStatus.calendarConfigured) {
            await addStep(procId, "Googleカレンダーに登録");
            try {
              await syncBookingToCalendar(match.name, parsed.start_date, parsed.end_date, bookingOwner);
              calendarNote = " Googleカレンダーにも登録しました。";
            } catch (e) {
              console.error("HORAI AI: calendar sync failed ->", e);
              calendarNote = " なお、Googleカレンダーへの登録には失敗しました（予約自体はHORAI AI内に登録済みです）。";
            }
          }

          pushAI(`${match.name}を${formatSlashRange(parsed.start_date, parsed.end_date)}で予約しました。担当は${bookingOwner}さんです。${calendarNote}`);
          setTimeout(() => setScreen("vehicles"), 500);
          setTimeout(() => setFlashBookingId(null), 3000);
          break;
        }

        case "create_case": {
          if (!parsed.case_name) {
            pushAI("登録する案件名を教えてください。");
            break;
          }
          await addStep(procId, "案件管理を確認");
          await addStep(procId, "案件を作成・保存");
          const cid = `c-${Date.now()}`;
          const status = ALLOWED_STATUSES.includes(parsed.status) ? parsed.status : "見積中";
          const owner = parsed.owner || CURRENT_USER;
          setCases((cs) => [
            { id: cid, name: parsed.case_name, owner, status, sales: 0, profit: 0, deadline: addDays(getJapanDateString(), 30) },
            ...cs,
          ]);
          setFlashCaseId(cid);
          pushAI(`新規案件「${parsed.case_name}」を登録しました。担当は${owner}さん、ステータスは${status}です。`);
          setTimeout(() => setScreen("cases"), 500);
          setTimeout(() => setFlashCaseId(null), 3000);
          break;
        }

        case "list_deadline_cases": {
          await addStep(procId, "案件を分析");
          const today = getJapanDateString();
          const soon = cases
            .filter((c) => {
              const days = (new Date(`${c.deadline}T00:00:00+09:00`) - new Date(`${today}T00:00:00+09:00`)) / 86400000;
              return days >= 0 && days <= 14;
            })
            .sort((a, b) => (a.deadline < b.deadline ? -1 : 1));
          setChatLog((l) => [...l, { id: `res-${Date.now()}`, type: "results", cases: soon }]);
          pushAI(soon.length ? `見積期限が近い案件が${soon.length}件あります。` : "現在、期限が近い案件はありません。");
          setScreen("home");
          break;
        }

        case "notify": {
          if (!parsed.assignee) {
            pushAI("通知する担当者名を教えてください。");
            break;
          }
          if (!parsed.message) {
            pushAI("通知する内容を教えてください。");
            break;
          }
          await addStep(procId, "宛先を確認");
          await addStep(procId, "通知を送信");
          setNotifications((n) => [{ id: `n-${Date.now()}`, to: parsed.assignee, text: parsed.message, time: "たった今" }, ...n]);
          setUnread((u) => u + 1);
          pushAI(`${parsed.assignee}さんに通知を送信しました。`);
          break;
        }

        case "sales_summary": {
          await addStep(procId, "データを集計");
          setSalesReady(true);
          pushAI(parsed.reply || "今月の営業売上をまとめました。");
          setTimeout(() => setScreen("sales"), 500);
          break;
        }

        case "list_cases": {
          pushAI(parsed.reply || "案件一覧を開きます。");
          setTimeout(() => setScreen("cases"), 400);
          break;
        }

        case "list_vehicles": {
          pushAI(parsed.reply || "車両管理を開きます。");
          setTimeout(() => setScreen("vehicles"), 400);
          break;
        }

        default: {
          pushAI(parsed.reply || "このデモでは「車両予約」「案件登録」「見積期限の確認」「通知依頼」「売上集計」に対応しています。");
        }
      }
    } finally {
      setIsBusy(false);
    }
  };

  const openMore = (key) => {
    setSubScreen(key);
    setMoreOpen(false);
  };

  return (
    <div className="w-full h-screen flex items-center justify-center" style={{ background: "#DDE3EC", fontFamily: "'Inter', sans-serif" }}>
      <div
        className="relative flex flex-col overflow-hidden"
        style={{
          width: 390,
          height: "min(844px, 96vh)",
          background: "#F6F8FC",
          borderRadius: 40,
          border: "10px solid #0B1120",
          boxShadow: "0 30px 60px rgba(11,31,61,0.25)",
        }}
      >
        <div className="absolute top-0 left-1/2 -translate-x-1/2 w-32 h-6 bg-[#0B1120] rounded-b-2xl z-30" />

        <TopBar
          screen={subScreen || screen}
          unread={unread}
          onBack={subScreen ? () => setSubScreen(null) : null}
          onBell={() => openMore("notifications")}
        />

        <div className="flex-1 overflow-hidden relative">
          {subScreen === "employees" && <EmployeesScreen />}
          {subScreen === "notifications" && <NotificationsScreen notifications={notifications} />}
          {subScreen === "settings" && (
            <SettingsScreen
              mockMode={mockMode}
              presentationMode={presentationMode}
              setPresentationMode={setPresentationMode}
              googleStatus={googleStatus}
              autoExportSheets={autoExportSheets}
              setAutoExportSheets={setAutoExportSheets}
              sheetsExport={sheetsExport}
              onExportNow={exportToSheets}
            />
          )}

          {!subScreen && screen === "home" && (
            <HomeScreen chatLog={chatLog} input={input} setInput={setInput} handleSubmit={handleSubmit} isBusy={isBusy} logEndRef={logEndRef} mockMode={mockMode} presentationMode={presentationMode} />
          )}
          {!subScreen && screen === "cases" && <CasesScreen cases={cases} flashCaseId={flashCaseId} />}
          {!subScreen && screen === "vehicles" && <VehiclesScreen vehicles={vehicles} flashBookingId={flashBookingId} />}
          {!subScreen && screen === "sales" && <SalesScreen ready={salesReady} />}
        </div>

        {!subScreen && (
          <BottomTabs
            screen={screen}
            setScreen={(k) => {
              if (k === "more") setMoreOpen(true);
              else setScreen(k);
            }}
            unread={unread}
          />
        )}

        {moreOpen && (
          <div className="absolute inset-0 z-40 flex flex-col justify-end">
            <div className="flex-1" onClick={() => setMoreOpen(false)} />
            <div className="bg-white rounded-t-3xl px-4 pt-3 pb-8" style={{ boxShadow: "0 -10px 30px rgba(11,31,61,0.15)" }}>
              <div className="w-10 h-1 rounded-full bg-gray-300 mx-auto mb-4" />
              {MORE_ITEMS.map(({ key, label, icon: Icon }) => (
                <button
                  key={key}
                  onClick={() => openMore(key)}
                  className="w-full flex items-center gap-3 px-2 py-3.5 text-[14px] border-b last:border-0"
                  style={{ borderColor: "#F1F3F8", color: NAVY }}
                >
                  <div className="w-9 h-9 rounded-xl flex items-center justify-center" style={{ background: "#EAF1FF" }}>
                    <Icon size={16} color={SKY} />
                  </div>
                  {label}
                  {key === "notifications" && unread > 0 && (
                    <span className="ml-auto text-[10px] font-bold rounded-full w-4 h-4 flex items-center justify-center" style={{ background: "#FF5C5C", color: "#fff" }}>
                      {unread}
                    </span>
                  )}
                  <ChevronRight size={15} style={{ marginLeft: 8, color: "#B7C2D6" }} />
                </button>
              ))}
            </div>
          </div>
        )}
      </div>
    </div>
  );
}

/* ------------------------------------------------------------------ */
/*  top bar / bottom tabs                                               */
/* ------------------------------------------------------------------ */

const TITLES = { home: "HORAI AI", cases: "案件管理", vehicles: "車両管理", sales: "売上管理", employees: "社員一覧", notifications: "通知", settings: "設定" };

function TopBar({ screen, unread, onBack, onBell }) {
  return (
    <div className="pt-9 pb-3 px-4 flex items-center gap-2 shrink-0 z-20" style={{ background: NAVY, color: "#fff" }}>
      {onBack ? (
        <button onClick={onBack} className="w-7 h-7 flex items-center justify-center -ml-1">
          <ChevronLeft size={19} />
        </button>
      ) : (
        <div className="w-7 h-7 rounded-lg flex items-center justify-center" style={{ background: `linear-gradient(135deg, ${SKY}, #7DB2FF)` }}>
          <Sparkles size={14} color="#fff" />
        </div>
      )}
      <div className="text-[15px] font-bold" style={{ fontFamily: "'Sora', sans-serif" }}>{TITLES[screen] || "HORAI AI"}</div>
      <button onClick={onBell} className="ml-auto relative w-8 h-8 flex items-center justify-center">
        <Bell size={17} color="rgba(255,255,255,0.85)" />
        {unread > 0 && <span className="absolute top-1 right-1 w-2.5 h-2.5 rounded-full" style={{ background: "#FF5C5C", border: "1.5px solid " + NAVY }} />}
      </button>
    </div>
  );
}

function BottomTabs({ screen, setScreen, unread }) {
  return (
    <div className="shrink-0 flex items-stretch px-1 pt-1.5 z-20" style={{ background: "#fff", borderTop: "1px solid #E6EAF1", paddingBottom: "env(safe-area-inset-bottom, 10px)" }}>
      {TABS.map(({ key, label, icon: Icon }) => {
        const active = screen === key;
        return (
          <button key={key} onClick={() => setScreen(key)} className="flex-1 flex flex-col items-center gap-1 py-1.5 relative">
            <Icon size={19} strokeWidth={2} color={active ? SKY : "#9AA5BD"} />
            <span className="text-[10px]" style={{ color: active ? SKY : "#9AA5BD", fontWeight: active ? 700 : 500 }}>{label}</span>
            {key === "more" && unread > 0 && <span className="absolute top-0 right-6 w-2 h-2 rounded-full" style={{ background: "#FF5C5C" }} />}
          </button>
        );
      })}
    </div>
  );
}

/* ------------------------------------------------------------------ */
/*  home / chat                                                         */
/* ------------------------------------------------------------------ */

function HomeScreen({ chatLog, input, setInput, handleSubmit, isBusy, logEndRef, mockMode, presentationMode }) {
  const suggestions = ["ハイエースを明日から2日間借りたい", "新規案件でテルメ工業を登録", "見積期限が近い案件は？", "池田さんに焼肉でんの確認を依頼", "今月の売上をまとめて"];

  return (
    <div className="h-full flex flex-col">
      {mockMode === true && !presentationMode && (
        <div className="mx-3.5 mt-3 mb-0.5 rounded-xl px-3 py-2 text-[10.5px] flex items-center gap-2" style={{ background: "rgba(227,160,8,0.12)", color: "#8A6400", border: "1px solid rgba(227,160,8,0.3)" }}>
          <AlertTriangle size={12} />
          モックモードで動作中です（APIキー未設定 / 簡易ルールで解析しています）
        </div>
      )}
      <div className="flex-1 overflow-y-auto px-3.5 pt-3.5 pb-2">
        <div className="flex flex-col gap-2.5">
          {chatLog.map((m) => {
            if (m.type === "process") return <ProcessCard key={m.id} steps={m.steps} />;
            if (m.type === "results")
              return (
                <div key={m.id} className="flex flex-col gap-2">
                  {m.cases.length === 0 && <div className="text-[11.5px] px-1" style={{ color: "#9AA5BD" }}>該当する案件はありません</div>}
                  {m.cases.map((c) => (
                    <div key={c.id} className="rounded-xl p-3 flex items-center gap-2.5" style={{ background: "#fff", border: "1px solid #E6EAF1" }}>
                      <div className="w-8 h-8 rounded-lg flex items-center justify-center shrink-0" style={{ background: "rgba(227,160,8,0.14)" }}>
                        <CalendarDays size={14} color={WARN} />
                      </div>
                      <div>
                        <div className="text-[13px] font-semibold" style={{ color: NAVY }}>{c.name}</div>
                        <div className="text-[10.5px]" style={{ color: "#9AA5BD" }}>担当 {c.owner} ・ 期限 {md(c.deadline)}</div>
                      </div>
                    </div>
                  ))}
                </div>
              );
            return <ChatBubble key={m.id} role={m.role} text={m.text} />;
          })}
          <div ref={logEndRef} />
        </div>
      </div>

      {!isBusy && chatLog.length < 3 && (
        <div className="flex gap-2 px-3.5 pb-2 overflow-x-auto no-scrollbar">
          {suggestions.map((s) => (
            <button key={s} onClick={() => setInput(s)} className="shrink-0 text-[11px] px-3 py-1.5 rounded-full whitespace-nowrap" style={{ border: "1px solid #E6EAF1", color: "#6B7690", background: "#fff" }}>
              {s}
            </button>
          ))}
        </div>
      )}

      <div className="px-3 pb-3 pt-1 shrink-0">
        <div className="rounded-2xl border flex items-center gap-2 px-3 py-2.5" style={{ borderColor: isBusy ? SKY : "#E6EAF1", background: "#fff", boxShadow: "0 6px 20px rgba(11,31,61,0.06)" }}>
          <div className="w-7 h-7 rounded-full flex items-center justify-center shrink-0" style={{ background: "linear-gradient(135deg,#4C8DFF,#8AB6FF)" }}>
            <Sparkles size={12} color="#fff" />
          </div>
          <input
            value={input}
            onChange={(e) => setInput(e.target.value)}
            onKeyDown={(e) => e.key === "Enter" && handleSubmit()}
            placeholder={isBusy ? "AIが考えています…" : "業務内容を入力"}
            maxLength={2000}
            className="flex-1 outline-none text-[13px] bg-transparent min-w-0"
            disabled={isBusy}
          />
          <button onClick={handleSubmit} disabled={isBusy || !input.trim()} className="w-8 h-8 rounded-xl flex items-center justify-center shrink-0" style={{ background: NAVY, opacity: isBusy || !input.trim() ? 0.35 : 1 }}>
            {isBusy ? <Loader2 size={13} color="#fff" style={{ animation: "horai-spin 0.9s linear infinite" }} /> : <Send size={13} color="#fff" />}
          </button>
        </div>
      </div>
      <style>{`.no-scrollbar::-webkit-scrollbar{display:none} @keyframes horai-spin { to { transform: rotate(360deg); } }`}</style>
    </div>
  );
}

function ChatBubble({ role, text }) {
  const isAI = role === "ai";
  return (
    <div className={`flex ${isAI ? "justify-start" : "justify-end"}`}>
      <div className="max-w-[84%] rounded-2xl px-3.5 py-2.5 text-[12.5px] leading-relaxed" style={isAI ? { background: "#fff", border: "1px solid #E6EAF1", color: NAVY } : { background: NAVY, color: "#fff" }}>
        {text}
      </div>
    </div>
  );
}

function ProcessCard({ steps }) {
  const hasError = steps.some((s) => s.status === "error");
  const allDone = steps.every((s) => s.status === "done");
  return (
    <div className="rounded-2xl p-3.5" style={{ background: "rgba(255,255,255,0.75)", border: `1px solid ${hasError ? DANGER : "#E6EAF1"}`, backdropFilter: "blur(10px)" }}>
      <div className="flex items-center gap-2 mb-2.5">
        <div className="relative w-6 h-6 flex items-center justify-center">
          {!hasError && (
            <>
              <div className="absolute inset-0 rounded-full" style={{ background: `conic-gradient(${SKY}, #8AB6FF, ${SKY})`, opacity: allDone ? 0.25 : 0.9, animation: allDone ? "none" : "horai-spin 2s linear infinite" }} />
              <div className="absolute rounded-full" style={{ inset: 2.5, background: "#fff" }} />
              <Sparkles size={11} color={allDone ? "#B7C2D6" : SKY} className="relative" />
            </>
          )}
          {hasError && <AlertTriangle size={16} color={DANGER} />}
        </div>
        <span className="text-[11px] font-semibold" style={{ color: hasError ? DANGER : "#6B7690" }}>
          {hasError ? "処理に失敗しました" : allDone ? "処理が完了しました" : "AIが処理を実行しています…"}
        </span>
      </div>
      <div className="flex flex-col gap-1.5">
        {steps.map((s, i) => (
          <div key={i} className="flex items-center gap-2">
            {s.status === "done" && (
              <div className="w-4 h-4 rounded-full flex items-center justify-center shrink-0" style={{ background: OK }}>
                <Check size={10} color="#fff" strokeWidth={3} />
              </div>
            )}
            {s.status === "active" && <Loader2 size={15} style={{ color: SKY, animation: "horai-spin 0.9s linear infinite" }} />}
            {s.status === "pending" && <Circle size={13} color="#DCE2ED" />}
            {s.status === "error" && (
              <div className="w-4 h-4 rounded-full flex items-center justify-center shrink-0" style={{ background: DANGER }}>
                <X size={10} color="#fff" strokeWidth={3} />
              </div>
            )}
            <span className="text-[11.5px]" style={{ color: s.status === "pending" ? "#B7C2D6" : s.status === "error" ? DANGER : NAVY, fontWeight: s.status === "active" ? 600 : 500 }}>
              {s.label}
            </span>
          </div>
        ))}
      </div>
    </div>
  );
}

/* ------------------------------------------------------------------ */
/*  cases                                                                */
/* ------------------------------------------------------------------ */

function CasesScreen({ cases, flashCaseId }) {
  const [q, setQ] = useState("");
  const filtered = cases.filter((c) => c.name.includes(q));
  return (
    <div className="h-full overflow-y-auto px-3.5 py-3.5">
      <div className="flex items-center gap-2 px-3 py-2 rounded-xl mb-3" style={{ background: "#fff", border: "1px solid #E6EAF1" }}>
        <Search size={13} color="#9AA5BD" />
        <input value={q} onChange={(e) => setQ(e.target.value)} placeholder="案件名で検索" className="text-[12px] outline-none bg-transparent flex-1" />
      </div>
      <div className="flex flex-col gap-2.5">
        {filtered.map((c) => (
          <div key={c.id} className="rounded-2xl p-3.5 transition-all duration-500" style={{ background: "#fff", border: `1px solid ${flashCaseId === c.id ? SKY : "#E6EAF1"}`, boxShadow: flashCaseId === c.id ? "0 0 0 3px rgba(76,141,255,0.14)" : "none" }}>
            <div className="flex items-center justify-between mb-2">
              <div className="font-semibold text-[13.5px]" style={{ color: NAVY }}>{c.name}</div>
              <span className="text-[10px] font-semibold px-2 py-0.5 rounded-full" style={{ background: `${STATUS_COLOR[c.status] || SKY}18`, color: STATUS_COLOR[c.status] || SKY }}>{c.status}</span>
            </div>
            <div className="text-[11px] mb-2" style={{ color: "#6B7690" }}>担当：{c.owner} ・ 期限 {md(c.deadline)}</div>
            <div className="flex gap-4 text-[11.5px]">
              <div><span style={{ color: "#9AA5BD" }}>売上 </span><span className="font-mono font-semibold" style={{ color: NAVY }}>{yen(c.sales)}</span></div>
              <div><span style={{ color: "#9AA5BD" }}>粗利 </span><span className="font-mono font-semibold" style={{ color: NAVY }}>{yen(c.profit)}</span></div>
            </div>
          </div>
        ))}
      </div>
    </div>
  );
}

/* ------------------------------------------------------------------ */
/*  vehicles                                                             */
/* ------------------------------------------------------------------ */

function VehiclesScreen({ vehicles, flashBookingId }) {
  return (
    <div className="h-full overflow-y-auto px-3.5 py-3.5">
      <div className="flex flex-col gap-3">
        {vehicles.map((v) => (
          <div key={v.id} className="rounded-2xl p-3.5" style={{ background: "#fff", border: "1px solid #E6EAF1" }}>
            <div className="flex items-center gap-2.5 mb-2.5">
              <div className="w-9 h-9 rounded-xl flex items-center justify-center" style={{ background: "#EAF1FF" }}>
                <Car size={16} color={SKY} />
              </div>
              <div>
                <div className="font-semibold text-[13px]" style={{ color: NAVY }}>{v.name}</div>
                <div className="text-[10px]" style={{ color: "#9AA5BD" }}>{v.plate}</div>
              </div>
              <span className="ml-auto text-[10px] font-semibold px-2 py-0.5 rounded-full" style={{ background: v.bookings.length ? "rgba(227,160,8,0.14)" : "rgba(47,179,128,0.14)", color: v.bookings.length ? WARN : OK }}>
                {v.bookings.length ? "予約あり" : "空車"}
              </span>
            </div>
            <div className="flex flex-col gap-1.5">
              {v.bookings.length === 0 && <div className="text-[11px] py-2.5 text-center rounded-lg" style={{ color: "#B7C2D6", background: "#F6F8FC" }}>予約はありません</div>}
              {v.bookings.map((b) => (
                <div key={b.id} className="flex items-center gap-2 rounded-lg px-2.5 py-2 transition-all duration-500" style={{ background: flashBookingId === b.id ? "rgba(76,141,255,0.09)" : "#F6F8FC", border: `1px solid ${flashBookingId === b.id ? SKY : "transparent"}` }}>
                  <CalendarDays size={12} color="#6B7690" />
                  <span className="text-[11.5px] font-mono" style={{ color: NAVY }}>{formatSlashRange(b.startDate, b.endDate)}</span>
                  <span className="ml-auto text-[11px]" style={{ color: "#6B7690" }}>{b.who}</span>
                </div>
              ))}
            </div>
          </div>
        ))}
      </div>
    </div>
  );
}

/* ------------------------------------------------------------------ */
/*  sales                                                                */
/* ------------------------------------------------------------------ */

function SalesScreen({ ready }) {
  const total = SALES_BY_REP.reduce((a, r) => a + r.売上, 0);
  const profit = SALES_BY_REP.reduce((a, r) => a + r.粗利, 0);
  const pieData = SALES_BY_REP.map((r) => ({ name: r.name, value: r.売上 }));

  return (
    <div className="h-full overflow-y-auto px-3.5 py-3.5">
      <div className="text-[10.5px] mb-3 px-1" style={{ color: "#9AA5BD" }}>現在はプレゼン用のサンプルデータです。</div>
      <div className="grid grid-cols-2 gap-2.5 mb-3">
        <StatCard label="今月売上合計" value={total} format={yen} icon={TrendingUp} up />
        <StatCard label="粗利合計" value={profit} format={yen} icon={ArrowUpRight} up />
      </div>
      <div className="rounded-2xl p-3.5 mb-3" style={{ background: "#fff", border: "1px solid #E6EAF1" }}>
        <div className="text-[12px] font-semibold mb-2" style={{ color: NAVY }}>営業担当別 売上</div>
        <div style={{ width: "100%", height: 170 }}>
          <ResponsiveContainer>
            <BarChart data={SALES_BY_REP}>
              <CartesianGrid strokeDasharray="3 3" stroke="#EEF1F6" vertical={false} />
              <XAxis dataKey="name" tick={{ fontSize: 10, fill: "#6B7690" }} axisLine={{ stroke: "#E6EAF1" }} tickLine={false} />
              <YAxis tick={{ fontSize: 9, fill: "#9AA5BD" }} axisLine={false} tickLine={false} tickFormatter={(v) => `${v / 1000000}M`} width={30} />
              <Tooltip formatter={(v) => yen(v)} contentStyle={{ fontSize: 11, borderRadius: 10, border: "1px solid #E6EAF1" }} />
              <Bar dataKey="売上" fill={SKY} radius={[5, 5, 0, 0]} isAnimationActive={ready} />
            </BarChart>
          </ResponsiveContainer>
        </div>
      </div>
      <div className="rounded-2xl p-3.5 mb-3" style={{ background: "#fff", border: "1px solid #E6EAF1" }}>
        <div className="text-[12px] font-semibold mb-2" style={{ color: NAVY }}>売上構成比</div>
        <div style={{ width: "100%", height: 190 }}>
          <ResponsiveContainer>
            <PieChart>
              <Pie data={pieData} dataKey="value" nameKey="name" innerRadius={42} outerRadius={68} isAnimationActive={ready}>
                {pieData.map((_, i) => <Cell key={i} fill={PIE_COLORS[i % PIE_COLORS.length]} />)}
              </Pie>
              <Tooltip formatter={(v) => yen(v)} contentStyle={{ fontSize: 11, borderRadius: 10, border: "1px solid #E6EAF1" }} />
            </PieChart>
          </ResponsiveContainer>
        </div>
      </div>
      <div className="rounded-2xl overflow-hidden" style={{ background: "#fff", border: "1px solid #E6EAF1" }}>
        {SALES_BY_REP.map((r, i) => (
          <div key={r.name} className="flex items-center justify-between px-3.5 py-2.5 text-[12px]" style={{ borderTop: i ? "1px solid #F1F3F8" : "none" }}>
            <span style={{ color: NAVY }}>{r.name}</span>
            <span className="font-mono">{yen(r.売上)}</span>
            <span className="font-mono" style={{ color: OK }}>{((r.粗利 / r.売上) * 100).toFixed(1)}%</span>
          </div>
        ))}
      </div>
    </div>
  );
}

function StatCard({ label, value, format, icon: Icon, up }) {
  return (
    <div className="rounded-2xl p-3" style={{ background: "#fff", border: "1px solid #E6EAF1" }}>
      <div className="flex items-center justify-between mb-1.5">
        <div className="text-[10.5px]" style={{ color: "#9AA5BD" }}>{label}</div>
        <div className="w-6 h-6 rounded-lg flex items-center justify-center" style={{ background: "#EAF1FF" }}>
          <Icon size={11} color={SKY} />
        </div>
      </div>
      <div className="text-[15px] font-bold font-mono" style={{ color: NAVY }}><CountUp to={value} format={format} /></div>
      <div className="flex items-center gap-1 mt-1 text-[10px]" style={{ color: up ? OK : "#E36A6A" }}>
        {up ? <ArrowUpRight size={10} /> : <ArrowDownRight size={10} />} 前月比 +8.4%
      </div>
    </div>
  );
}

/* ------------------------------------------------------------------ */
/*  employees / notifications / settings                                */
/* ------------------------------------------------------------------ */

function EmployeesScreen() {
  return (
    <div className="h-full overflow-y-auto px-3.5 py-3.5">
      <div className="grid grid-cols-2 gap-2.5">
        {EMPLOYEES.map((e) => (
          <div key={e.id} className="rounded-2xl p-3 flex flex-col items-center text-center" style={{ background: "#fff", border: "1px solid #E6EAF1" }}>
            <div className="w-11 h-11 rounded-full flex items-center justify-center mb-2 text-[14px] font-bold" style={{ background: `linear-gradient(135deg, ${SKY}, ${NAVY_SOFT})`, color: "#fff", fontFamily: "'Sora', sans-serif" }}>
              {e.name.slice(0, 1)}
            </div>
            <div className="font-semibold text-[12.5px]" style={{ color: NAVY }}>{e.name}</div>
            <div className="text-[10px] mt-0.5" style={{ color: "#9AA5BD" }}>{e.dept}</div>
            <div className="mt-2 text-[10px] px-2 py-0.5 rounded-full" style={{ background: "#F6F8FC", color: "#6B7690" }}>担当 {e.cases} 件</div>
          </div>
        ))}
      </div>
    </div>
  );
}

function NotificationsScreen({ notifications }) {
  return (
    <div className="h-full overflow-y-auto px-3.5 py-3.5">
      {notifications.length === 0 ? (
        <div className="rounded-2xl p-8 text-center" style={{ background: "#fff", border: "1px dashed #E6EAF1", color: "#B7C2D6" }}>
          <Bell size={22} className="mx-auto mb-2" />
          <div className="text-[12px]">まだ通知はありません</div>
        </div>
      ) : (
        <div className="flex flex-col gap-2">
          {notifications.map((n) => (
            <div key={n.id} className="rounded-xl p-3 flex items-start gap-2.5" style={{ background: "#fff", border: "1px solid #E6EAF1" }}>
              <div className="w-8 h-8 rounded-full flex items-center justify-center shrink-0" style={{ background: "#EAF1FF" }}>
                <Bell size={13} color={SKY} />
              </div>
              <div>
                <div className="text-[12px]" style={{ color: NAVY }}><span className="font-semibold">{n.to}さん宛　</span>{n.text}</div>
                <div className="text-[10px] mt-0.5" style={{ color: "#9AA5BD" }}>{n.time}</div>
              </div>
            </div>
          ))}
        </div>
      )}
    </div>
  );
}

function SettingsScreen({ mockMode, presentationMode, setPresentationMode, googleStatus, autoExportSheets, setAutoExportSheets, sheetsExport, onExportNow }) {
  const connectionValue = presentationMode
    ? "プレゼンモード（外部通信なしで動作中）"
    : mockMode === true
    ? "モックモード（APIキー未設定 / 簡易ルールで解析中）"
    : mockMode === false
    ? "Next.jsサーバー経由でClaude APIに接続中"
    : "Next.jsサーバー経由でClaude APIに接続（未確認：ホームでAIに話しかけると判明します）";
  const rows = [
    { label: "会社名", value: "宝来社" },
    { label: "接続方式", value: connectionValue },
    { label: "データ保存", value: "現在はこのブラウザ内に保存されます" },
    { label: "デモ状態", value: "プレゼン用デモ版" },
  ];

  const exportStatusText =
    sheetsExport.status === "loading"
      ? "書き出し中…"
      : sheetsExport.status === "success"
      ? `書き出し済み（${sheetsExport.time ? sheetsExport.time.toLocaleTimeString("ja-JP") : ""}）`
      : sheetsExport.status === "error"
      ? sheetsExport.message || "書き出しに失敗しました"
      : "";

  return (
    <div className="h-full overflow-y-auto px-3.5 py-3.5">
      <div className="rounded-2xl overflow-hidden mb-3" style={{ background: "#fff", border: "1px solid #E6EAF1" }}>
        {rows.map((r, i) => (
          <div key={r.label} className="flex flex-col gap-0.5 px-3.5 py-3" style={{ borderTop: i ? "1px solid #F1F3F8" : "none" }}>
            <span className="text-[10.5px]" style={{ color: "#9AA5BD" }}>{r.label}</span>
            <span className="text-[12.5px]" style={{ color: NAVY, fontWeight: 500 }}>{r.value}</span>
          </div>
        ))}
      </div>

      <div className="rounded-2xl p-3.5 mb-3" style={{ background: "#fff", border: "1px solid #E6EAF1" }}>
        <ToggleRow
          title="プレゼンモード"
          on={presentationMode}
          onToggle={() => setPresentationMode((v) => !v)}
          description="オンにすると、外部のAI通信を行わず、ブラウザ内だけで依頼内容を解析します。通信エラーやAPI障害の影響を受けないため、社内発表など「確実に動かしたい」場面向けです。文章の理解力は本来のClaude接続より簡易的になります。"
        />
      </div>

      <div className="rounded-2xl p-3.5 mb-3" style={{ background: "#fff", border: "1px solid #E6EAF1" }}>
        <div className="text-[12.5px] font-semibold mb-2.5" style={{ color: NAVY }}>Google連携</div>

        <div className="flex items-center justify-between py-1.5">
          <div>
            <div className="text-[12px]" style={{ color: NAVY }}>Googleカレンダー</div>
            <div className="text-[10.5px]" style={{ color: "#9AA5BD" }}>車両予約が成立すると自動で登録されます</div>
          </div>
          <StatusPill ok={googleStatus.calendarConfigured} />
        </div>

        <div className="h-px my-2.5" style={{ background: "#F1F3F8" }} />

        <div className="flex items-center justify-between py-1.5 mb-1">
          <div>
            <div className="text-[12px]" style={{ color: NAVY }}>Googleスプレッドシート</div>
            <div className="text-[10.5px]" style={{ color: "#9AA5BD" }}>案件・予約・通知をシートへ書き出します</div>
          </div>
          <StatusPill ok={googleStatus.sheetsConfigured} />
        </div>

        {googleStatus.sheetsConfigured && (
          <>
            <div className="flex items-center gap-2 mt-2">
              <button
                onClick={onExportNow}
                disabled={sheetsExport.status === "loading"}
                className="text-[11.5px] px-3 py-1.5 rounded-full font-medium"
                style={{ background: NAVY, color: "#fff", opacity: sheetsExport.status === "loading" ? 0.5 : 1 }}
              >
                今すぐ書き出す
              </button>
              {exportStatusText && (
                <span className="text-[10.5px]" style={{ color: sheetsExport.status === "error" ? DANGER : "#6B7690" }}>
                  {exportStatusText}
                </span>
              )}
            </div>
            <div className="mt-2.5">
              <ToggleRow
                title="自動で書き出す"
                small
                on={autoExportSheets}
                onToggle={() => setAutoExportSheets((v) => !v)}
                description="オンにすると、データが変わるたびに自動でスプレッドシートへ反映されます。"
              />
            </div>
          </>
        )}

        {!googleStatus.calendarConfigured && !googleStatus.sheetsConfigured && (
          <p className="text-[10.5px] leading-relaxed mt-1" style={{ color: "#9AA5BD" }}>
            未設定です。READMEの「Google連携」の手順に沿って環境変数を設定すると有効になります。
          </p>
        )}
      </div>
    </div>
  );
}

function StatusPill({ ok }) {
  return (
    <span
      className="text-[10px] font-semibold px-2 py-0.5 rounded-full shrink-0"
      style={{ background: ok ? "rgba(47,179,128,0.14)" : "rgba(154,165,189,0.16)", color: ok ? OK : "#8993A8" }}
    >
      {ok ? "連携済み" : "未設定"}
    </span>
  );
}

function ToggleRow({ title, description, on, onToggle, small }) {
  return (
    <div>
      <div className="flex items-center justify-between mb-1.5">
        <div className={small ? "text-[11.5px] font-medium" : "text-[12.5px] font-semibold"} style={{ color: NAVY }}>{title}</div>
        <button
          onClick={onToggle}
          className="relative w-11 h-6 rounded-full transition-colors shrink-0"
          style={{ background: on ? SKY : "#DCE2ED" }}
        >
          <span
            className="absolute top-0.5 w-5 h-5 rounded-full bg-white transition-all"
            style={{ left: on ? 22 : 2, boxShadow: "0 1px 3px rgba(11,31,61,0.25)" }}
          />
        </button>
      </div>
      {description && (
        <p className="text-[11px] leading-relaxed" style={{ color: "#6B7690" }}>
          {description}
        </p>
      )}
    </div>
  );
}
