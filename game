import { createInitialState, createDefaultPlan, runYear, PROMOTERS, TICKET_LEVELS, PILLARS } from './festivalSimEngine.js';

let state = createInitialState({ seed: 777, attendees: 300, cashReserve: 40000 });

let plan = createDefaultPlan();
plan.ticketLevel = TICKET_LEVELS.MODERATE;
plan.promoter = PROMOTERS.JUNEBUG;
plan.contingency = 6000;
plan.tiers[PILLARS.FACILITIES] = 4;
plan.tiers[PILLARS.MUSIC] = 3;

const { nextState, report } = runYear(state, plan);
console.log(report.lines.join('\n'));
state = nextState;
// src/modes/runFestival/festivalSimEngine.js
// 🍧 Run The Festival — Economy + Drain + Personality Sim Engine
// Pure logic module: no DOM, no storage, no MobX required.
// Your UI layer should:
//   1) keep state (from this engine)
//   2) build a "plan" object each year
//   3) call runYear(state, plan) -> { nextState, report }

export const PILLARS = Object.freeze({
  MUSIC: 'music',
  CONCESSIONS: 'concessions',
  FACILITIES: 'facilities',
  LESSONS: 'lessons',
  SECURITY: 'security',
});

export const PILLAR_LIST = Object.freeze([
  PILLARS.MUSIC,
  PILLARS.CONCESSIONS,
  PILLARS.FACILITIES,
  PILLARS.LESSONS,
  PILLARS.SECURITY,
]);

// Ticket levels: simple discrete choices (per your request).
export const TICKET_LEVELS = Object.freeze({
  CHEAP: 'cheap',
  MODERATE: 'moderate',
  EXPENSIVE: 'expensive',
});

export const TICKET_PRICE = Object.freeze({
  [TICKET_LEVELS.CHEAP]: 200,
  [TICKET_LEVELS.MODERATE]: 250,
  [TICKET_LEVELS.EXPENSIVE]: 320,
});

// Price affects: (1) how mad people are, (2) how many new people bite.
const PRICE_SAT_MULT = Object.freeze({
  [TICKET_LEVELS.CHEAP]: 1.04,
  [TICKET_LEVELS.MODERATE]: 1.00,
  [TICKET_LEVELS.EXPENSIVE]: 0.92,
});

const PRICE_DEMAND_MULT = Object.freeze({
  [TICKET_LEVELS.CHEAP]: 1.12,
  [TICKET_LEVELS.MODERATE]: 1.00,
  [TICKET_LEVELS.EXPENSIVE]: 0.88,
});

// Promoters: bring X% of last year’s attendance as NEW potential fans,
// scaled by culture + price demand.
// (Costs come from cashReserve pre-festival.)
export const PROMOTERS = Object.freeze({
  NONE: 'none',
  JUNEBUG: 'junebug',
  MISTY: 'misty',
  BLAZE: 'blaze',
});

export const PROMOTER_META = Object.freeze({
  [PROMOTERS.NONE]:   { cost: 0,     pct: 0.00, name: 'No Promoter (Raw Dog Mode)' },
  [PROMOTERS.JUNEBUG]:{ cost: 8000,  pct: 0.15, name: 'Junebug Promotions (Sticky Flyers)' },
  [PROMOTERS.MISTY]:  { cost: 20000, pct: 0.25, name: 'Misty (The Portal Algorithm)' },
  [PROMOTERS.BLAZE]:  { cost: 35000, pct: 0.40, name: 'Blaze Media (Billboards + Drones)' },
});

// Tier costs per pillar (1 best .. 5 worst).
// These are scaled to feel playable with ~300–1600 attendees at $250–$320.
export const TIER_COSTS = Object.freeze({
  [PILLARS.MUSIC]:       Object.freeze({ 1: 32000, 2: 24000, 3: 17000, 4: 12000, 5: 7000 }),
  [PILLARS.CONCESSIONS]: Object.freeze({ 1: 20000, 2: 15000, 3: 11000, 4:  8000, 5: 4000 }),
  [PILLARS.FACILITIES]:  Object.freeze({ 1: 26000, 2: 19000, 3: 14000, 4: 10000, 5: 5000 }),
  [PILLARS.LESSONS]:     Object.freeze({ 1: 15000, 2: 11000, 3:  8000, 4:  6000, 5: 3000 }),
  [PILLARS.SECURITY]:    Object.freeze({ 1: 24000, 2: 18000, 3: 13000, 4:  9000, 5: 5000 }),
});

// Tier “performance” knobs.
// capacityMult = how much drain you can absorb before meltdowns.
// quality = how much it contributes to satisfaction/culture.
export const TIER_PERF = Object.freeze({
  1: Object.freeze({ capacityMult: 1.40, quality: 1.00, resilience: 1.25 }),
  2: Object.freeze({ capacityMult: 1.22, quality: 0.85, resilience: 1.12 }),
  3: Object.freeze({ capacityMult: 1.08, quality: 0.70, resilience: 1.00 }),
  4: Object.freeze({ capacityMult: 0.95, quality: 0.55, resilience: 0.90 }),
  5: Object.freeze({ capacityMult: 0.78, quality: 0.30, resilience: 0.78 }),
});

// “Culture grade” weights: what the crowd *cares about* overall.
export const CULTURE_WEIGHTS = Object.freeze({
  [PILLARS.MUSIC]: 0.25,
  [PILLARS.FACILITIES]: 0.25,
  [PILLARS.SECURITY]: 0.20,
  [PILLARS.CONCESSIONS]: 0.20,
  [PILLARS.LESSONS]: 0.10,
});

// Festival duration (drain simulation runs across days).
export const DEFAULT_FESTIVAL_DAYS = 3;

// Overhead: boring baseline costs that scale with attendance.
// (permits, base staffing, trash pickup, etc.)
export const DEFAULT_OVERHEAD = Object.freeze({
  fixed: 10000,
  perHead: 20,
});

// Loan: fixed 10-year payoff target. User can fail.
export const DEFAULT_LOAN = Object.freeze({
  total: 200000,
  annualPayment: 20000,
  years: 10,
});

// Investors/advertisers appear at certain years with “profit traps”.
export const DEFAULT_OFFER_YEARS = Object.freeze([2, 4, 6, 8, 9]);

// -----------------------------
// RNG (deterministic if you pass a seed)
// -----------------------------
export function mulberry32(seed) {
  let a = seed >>> 0;
  return function rng() {
    a |= 0;
    a = (a + 0x6D2B79F5) | 0;
    let t = Math.imul(a ^ (a >>> 15), 1 | a);
    t = (t + Math.imul(t ^ (t >>> 7), 61 | t)) ^ t;
    return ((t ^ (t >>> 14)) >>> 0) / 4294967296;
  };
}

export function randInt(rng, min, max) {
  return Math.floor(rng() * (max - min + 1)) + min;
}

export function pick(rng, arr) {
  return arr[Math.floor(rng() * arr.length)];
}

function clamp(n, lo, hi) {
  return Math.max(lo, Math.min(hi, n));
}

// -----------------------------
// Fan segments (personalities)
// -----------------------------
// Each segment has:
// - sizePct: fraction of crowd
// - prefs: weights per pillar (sum approx 1)
// - meltdownSensitivity: how hard they get hit by meltdowns
// - priceSensitivity: how much expensive tickets annoy them
export function buildDefaultSegments() {
  // Balanced but distinct cliques (fun + stable to tune).
  // Sum sizePct = 1.0
  return [
    {
      id: 'music-heads',
      label: 'Music-Heads',
      sizePct: 0.25,
      prefs: { music: 0.50, concessions: 0.15, facilities: 0.10, lessons: 0.05, security: 0.20 },
      meltdownSensitivity: { facilities: 0.55, security: 0.70, concessions: 0.40, lessons: 0.25, music: 0.30 },
      priceSensitivity: 0.40,
    },
    {
      id: 'foodies',
      label: 'Foodies',
      sizePct: 0.20,
      prefs: { music: 0.15, concessions: 0.50, facilities: 0.15, lessons: 0.05, security: 0.15 },
      meltdownSensitivity: { facilities: 0.65, security: 0.55, concessions: 0.80, lessons: 0.25, music: 0.35 },
      priceSensitivity: 0.55,
    },
    {
      id: 'comfort-crew',
      label: 'Comfort Crew',
      sizePct: 0.20,
      prefs: { music: 0.10, concessions: 0.15, facilities: 0.55, lessons: 0.05, security: 0.15 },
      meltdownSensitivity: { facilities: 1.00, security: 0.75, concessions: 0.50, lessons: 0.25, music: 0.25 },
      priceSensitivity: 0.65,
    },
    {
      id: 'math-goblins',
      label: 'Math Goblins',
      sizePct: 0.15,
      prefs: { music: 0.10, concessions: 0.10, facilities: 0.10, lessons: 0.55, security: 0.15 },
      meltdownSensitivity: { facilities: 0.55, security: 0.65, concessions: 0.40, lessons: 0.95, music: 0.25 },
      priceSensitivity: 0.45,
    },
    {
      id: 'safety-fams',
      label: 'Safety Families',
      sizePct: 0.20,
      prefs: { music: 0.10, concessions: 0.15, facilities: 0.20, lessons: 0.05, security: 0.50 },
      meltdownSensitivity: { facilities: 0.70, security: 1.00, concessions: 0.45, lessons: 0.25, music: 0.20 },
      priceSensitivity: 0.75,
    },
  ];
}

// -----------------------------
// State + Plan
// -----------------------------
export function createInitialState({
  seed = 1337,
  year = 1,
  attendees = 300,        // Misty “Year 1 pull” can be represented by setting this = 300.
  cashReserve = 40000,    // saved profits / startup grant / hooded dino seed money
  loanTotal = DEFAULT_LOAN.total,
  loanAnnualPayment = DEFAULT_LOAN.annualPayment,
  loanYears = DEFAULT_LOAN.years,
  festivalDays = DEFAULT_FESTIVAL_DAYS,
  overhead = DEFAULT_OVERHEAD,
  offerYears = DEFAULT_OFFER_YEARS,
  segments = buildDefaultSegments(),
} = {}) {
  return {
    seed,
    rng: null, // created on-demand each year for determinism
    year,
    attendees,
    cashReserve,
    lastYearProfit: 0,
    // Loan
    loanTotal,
    loanAnnualPayment,
    loanYears,
    loanRemaining: loanTotal,
    // Settings
    festivalDays,
    overhead,
    offerYears: Array.isArray(offerYears) ? offerYears.slice() : DEFAULT_OFFER_YEARS.slice(),
    // People
    segments,
    // Meta
    finished: false,
    farmPaidOff: false,
    // For story continuity:
    history: [],
  };
}

/**
 * Plan object you pass into runYear().
 * tiers: { music: 1..5, concessions: 1..5, facilities: 1..5, lessons: 1..5, security: 1..5 }
 * ticketLevel: cheap|moderate|expensive
 * promoter: none|junebug|misty|blaze
 * contingency: extra money set aside to patch meltdowns mid-festival (mostly used by facilities/security)
 * investorDealId: optional (if taking an offer)
 */
export function createDefaultPlan() {
  return {
    tiers: {
      [PILLARS.MUSIC]: 4,
      [PILLARS.CONCESSIONS]: 4,
      [PILLARS.FACILITIES]: 4,
      [PILLARS.LESSONS]: 4,
      [PILLARS.SECURITY]: 4,
    },
    ticketLevel: TICKET_LEVELS.MODERATE,
    promoter: PROMOTERS.NONE,
    contingency: 0,
    investorDealId: null,
  };
}

export function validatePlan(plan) {
  if (!plan || typeof plan !== 'object') return { ok: false, error: 'Plan missing' };
  if (!plan.tiers || typeof plan.tiers !== 'object') return { ok: false, error: 'Plan.tiers missing' };
  for (const p of PILLAR_LIST) {
    const t = plan.tiers[p];
    if (![1, 2, 3, 4, 5].includes(t)) return { ok: false, error: `Invalid tier for ${p}` };
  }
  if (!Object.values(TICKET_LEVELS).includes(plan.ticketLevel)) return { ok: false, error: 'Invalid ticketLevel' };
  if (!Object.values(PROMOTERS).includes(plan.promoter)) return { ok: false, error: 'Invalid promoter' };
  if (typeof plan.contingency !== 'number' || plan.contingency < 0) return { ok: false, error: 'Invalid contingency' };
  return { ok: true, error: null };
}

export function computeUpfrontCost(plan) {
  // Upfront costs are paid BEFORE the festival starts.
  const promoterCost = PROMOTER_META[plan.promoter]?.cost ?? 0;
  const tierSpend = PILLAR_LIST.reduce((sum, p) => sum + (TIER_COSTS[p][plan.tiers[p]] ?? 0), 0);
  const contingency = plan.contingency || 0;
  return {
    tierSpend,
    promoterCost,
    contingency,
    total: tierSpend + promoterCost + contingency,
  };
}

// -----------------------------
// Investor / advertiser offers (profit traps)
// -----------------------------
export function getOffersForYear(state) {
  const year = state.year;
  if (!state.offerYears.includes(year)) return [];

  // Offers are intentionally tempting: cash now, culture hit later.
  // Your UI can show these as goofy characters.
  return [
    {
      id: `offer-${year}-grease`,
      title: 'GreaseFund Capital',
      tagline: '“Cut corners, cash expands.”',
      cashNow: 25000,
      culturePenaltyNow: 0.05,
      culturePenaltyNext: 0.03,
      notes: 'Pure profit vibes; fans notice the soulless aura.',
    },
    {
      id: `offer-${year}-adspam`,
      title: 'Pop-Up Ad Wizard',
      tagline: '“We’ll market you… aggressively.”',
      cashNow: 15000,
      culturePenaltyNow: 0.02,
      culturePenaltyNext: 0.02,
      notes: 'Smaller cash; smaller cringe penalty.',
    },
    {
      id: `offer-${year}-sponsor`,
      title: 'MegaCorp Slush Co.',
      tagline: '“We own the vibes now.”',
      cashNow: 35000,
      culturePenaltyNow: 0.08,
      culturePenaltyNext: 0.05,
      notes: 'Big cash; big culture sellout hit.',
    },
  ];
}

function applyOfferIfAny(state, plan, rng) {
  const offers = getOffersForYear(state);
  if (!plan.investorDealId) {
    return { cashNow: 0, penaltyNow: 0, penaltyNext: 0, offerTaken: null };
  }

  const found = offers.find(o => o.id === plan.investorDealId);
  if (!found) return { cashNow: 0, penaltyNow: 0, penaltyNext: 0, offerTaken: null };

  // Optional randomness: sometimes the offer comes with a hidden “PR scandal” twist.
  const scandalRoll = rng();
  let penaltyNow = found.culturePenaltyNow;
  let penaltyNext = found.culturePenaltyNext;
  let cashNow = found.cashNow;

  if (scandalRoll < 0.10) {
    penaltyNow += 0.03;
    penaltyNext += 0.02;
  } else if (scandalRoll > 0.92) {
    cashNow += 5000;
  }

  return { cashNow, penaltyNow, penaltyNext, offerTaken: found };
}

// -----------------------------
// Pillar drain simulation
// -----------------------------
// Drain units per attendee per day (abstract units).
// You can tweak these later—this is where the magic lives.
const DRAIN_PER_HEAD_PER_DAY = Object.freeze({
  [PILLARS.MUSIC]: 1.00,        // crowd energy / stage demand
  [PILLARS.CONCESSIONS]: 1.35,  // food/water demand
  [PILLARS.FACILITIES]: 1.60,   // bathrooms/trash
  [PILLARS.LESSONS]: 0.55,      // lesson capacity demand
  [PILLARS.SECURITY]: 0.75,     // safety control demand
});

// How much of each segment actually participates/pressures each pillar.
// (Math goblins hit lessons harder, comfort crew hits facilities harder, etc.)
const SEGMENT_PRESSURE = Object.freeze({
  'music-heads':   { music: 1.25, concessions: 1.00, facilities: 0.85, lessons: 0.60, security: 0.90 },
  'foodies':       { music: 0.80, concessions: 1.35, facilities: 1.00, lessons: 0.60, security: 0.85 },
  'comfort-crew':  { music: 0.70, concessions: 0.90, facilities: 1.40, lessons: 0.55, security: 0.95 },
  'math-goblins':  { music: 0.70, concessions: 0.85, facilities: 0.80, lessons: 1.45, security: 0.85 },
  'safety-fams':   { music: 0.70, concessions: 0.95, facilities: 1.10, lessons: 0.60, security: 1.30 },
});

// Capacity baseline per attendee per day (units).
// Then tier capacityMult scales it.
const BASE_CAPACITY_PER_HEAD_PER_DAY = Object.freeze({
  [PILLARS.MUSIC]: 1.05,
  [PILLARS.CONCESSIONS]: 1.25,
  [PILLARS.FACILITIES]: 1.35,
  [PILLARS.LESSONS]: 0.60,
  [PILLARS.SECURITY]: 0.80,
});

// Meltdown thresholds and penalties.
const MELTDOWN_THRESH = Object.freeze({
  // If drain / capacity exceeds these -> meltdown tier.
  WARNING: 1.05,
  BAD: 1.18,
  DISASTER: 1.35,
});

// How much a meltdown hurts satisfaction across the crowd (base).
const MELTDOWN_PENALTY = Object.freeze({
  WARNING: 0.05,
  BAD: 0.12,
  DISASTER: 0.22,
});

// Contingency spending: patching can reduce meltdown severity.
// This is your “spend beyond cap to keep stuff running” mechanic.
const PATCH_COST = Object.freeze({
  // cost to downgrade the meltdown one level for that pillar for the whole festival
  WARNING_TO_OK: 4000,
  BAD_TO_WARNING: 6000,
  DISASTER_TO_BAD: 9000,
});

// -----------------------------
// Culture + satisfaction math
// -----------------------------
export function computeCultureGrade(tiers, culturePenalty = 0) {
  let c = 0;
  for (const p of PILLAR_LIST) {
    c += (CULTURE_WEIGHTS[p] || 0) * (TIER_PERF[tiers[p]]?.quality ?? 0.3);
  }
  c = clamp(c - culturePenalty, 0, 1);

  // Convert numeric culture to grade (for story flavor).
  let grade = 'C';
  if (c >= 0.90) grade = 'S';
  else if (c >= 0.80) grade = 'A';
  else if (c >= 0.70) grade = 'B';
  else if (c >= 0.60) grade = 'C';
  else if (c >= 0.50) grade = 'D';
  else grade = 'F';

  return { culture: c, grade };
}

// Segment satisfaction uses:
// - pillar quality (tiers)
// - pillar meltdowns (drain)
// - ticket price annoyance
export function computeSegmentSatisfaction(segment, tiers, meltdownByPillar, ticketLevel, culturePenalty = 0) {
  const priceMult = PRICE_SAT_MULT[ticketLevel] ?? 1.0;

  // Start from weighted pillar quality.
  let sat = 0;
  for (const p of PILLAR_LIST) {
    const w = segment.prefs[p] ?? 0;
    const q = TIER_PERF[tiers[p]]?.quality ?? 0.3;
    sat += w * q;
  }

  // Apply meltdown penalties (segment-specific sensitivity).
  let meltdownHit = 0;
  for (const p of PILLAR_LIST) {
    const m = meltdownByPillar[p]; // 'ok' | 'warning' | 'bad' | 'disaster'
    if (!m || m === 'ok') continue;

    const sens = segment.meltdownSensitivity[p] ?? 0.5;
    const basePenalty =
      m === 'warning' ? MELTDOWN_PENALTY.WARNING :
      m === 'bad' ? MELTDOWN_PENALTY.BAD :
      MELTDOWN_PENALTY.DISASTER;

    meltdownHit += sens * basePenalty;
  }

  // Price annoyance: expensive tickets annoy price-sensitive segments.
  // Cheap tickets can slightly reduce annoyance.
  const priceAnnoy =
    ticketLevel === TICKET_LEVELS.EXPENSIVE ? 0.10 * segment.priceSensitivity :
    ticketLevel === TICKET_LEVELS.CHEAP ? -0.03 * segment.priceSensitivity :
    0;

  sat = sat * priceMult;
  sat = sat - meltdownHit - priceAnnoy - culturePenalty;

  return clamp(sat, 0, 1);
}

// Return rate mapping: satisfaction -> probability of returning next year.
export function satisfactionToReturnRate(sat) {
  // Smooth curve: sat 0.2 => ~0.25 return, sat 0.6 => ~0.65, sat 0.85 => ~0.85
  const r = 0.10 + 0.90 * sat;
  return clamp(r, 0.20, 0.92);
}

// New fan pull: culture + flash (music) + price attractiveness.
export function computeWordOfMouthRate(culture, musicTier) {
  const flash = musicTier <= 2 ? 0.10 : (musicTier === 3 ? 0.06 : 0.03);
  const base = 0.08 + 0.20 * Math.max(0, culture - 0.50) + flash;
  return clamp(base, 0.06, 0.35);
}

// -----------------------------
// Drain sim core
// -----------------------------
function computeDrainForPillar(state, plan, pillar) {
  const tiers = plan.tiers;
  const days = state.festivalDays;
  const total = state.attendees;

  // Segment-weighted pressure:
  // sum over segments: (segment crowd) * (base drain/head/day) * (segment pressure multiplier) * days
  let drain = 0;
  for (const seg of state.segments) {
    const segCount = total * seg.sizePct;
    const base = DRAIN_PER_HEAD_PER_DAY[pillar] || 1.0;
    const pressure = (SEGMENT_PRESSURE[seg.id]?.[pillar] ?? 1.0);
    drain += segCount * base * pressure * days;
  }

  // Small randomness (weather, weird traffic, etc.) so it feels alive.
  // This is intentionally subtle so balance isn’t chaos.
  const rng = state.rng;
  const jitter = 0.96 + 0.10 * rng(); // 0.96..1.06
  drain *= jitter;

  return drain;
}

function computeCapacityForPillar(state, plan, pillar) {
  const tier = plan.tiers[pillar];
  const days = state.festivalDays;
  const total = state.attendees;

  const baseCap = BASE_CAPACITY_PER_HEAD_PER_DAY[pillar] || 1.0;
  const perf = TIER_PERF[tier] || TIER_PERF[5];
  const cap = total * baseCap * perf.capacityMult * days;

  return cap;
}

function classifyMeltdown(drain, capacity) {
  const ratio = capacity <= 0 ? 999 : drain / capacity;

  if (ratio <= MELTDOWN_THRESH.WARNING) return { level: 'ok', ratio };
  if (ratio <= MELTDOWN_THRESH.BAD) return { level: 'warning', ratio };
  if (ratio <= MELTDOWN_THRESH.DISASTER) return { level: 'bad', ratio };
  return { level: 'disaster', ratio };
}

function applyContingencyPatches(plan, meltdownByPillar, remainingContingency) {
  // Greedy patching: fix Facilities first (because toilets kill the vibe), then Security, then Concessions.
  const priority = [PILLARS.FACILITIES, PILLARS.SECURITY, PILLARS.CONCESSIONS, PILLARS.MUSIC, PILLARS.LESSONS];

  const patched = { ...meltdownByPillar };
  let cash = remainingContingency;

  function downgrade(pillar) {
    const cur = patched[pillar];
    if (cur === 'disaster') {
      if (cash >= PATCH_COST.DISASTER_TO_BAD) {
        cash -= PATCH_COST.DISASTER_TO_BAD;
        patched[pillar] = 'bad';
      }
    } else if (cur === 'bad') {
      if (cash >= PATCH_COST.BAD_TO_WARNING) {
        cash -= PATCH_COST.BAD_TO_WARNING;
        patched[pillar] = 'warning';
      }
    } else if (cur === 'warning') {
      if (cash >= PATCH_COST.WARNING_TO_OK) {
        cash -= PATCH_COST.WARNING_TO_OK;
        patched[pillar] = 'ok';
      }
    }
  }

  // Patch repeatedly, since downgrading one level might reveal you can downgrade again.
  // (Example: DISASTER -> BAD -> WARNING if you have cash.)
  for (let pass = 0; pass < 3; pass++) {
    for (const p of priority) {
      downgrade(p);
    }
  }

  return { patchedMeltdowns: patched, contingencyLeft: cash };
}

// -----------------------------
// Year simulation
// -----------------------------
export function runYear(state, plan) {
  if (state.finished) {
    return { nextState: state, report: { error: 'Game finished' } };
  }

  const v = validatePlan(plan);
  if (!v.ok) {
    return { nextState: state, report: { error: v.error } };
  }

  // Deterministic RNG per year
  const rng = mulberry32((state.seed + state.year * 99991) >>> 0);
  const workingState = { ...state, rng };

  // Upfront spending must be within cashReserve (profits saved).
  const cost = computeUpfrontCost(plan);
  if (cost.total > state.cashReserve) {
    return {
      nextState: state,
      report: {
        error: 'Not enough cashReserve for this plan.',
        needed: cost.total,
        have: state.cashReserve,
        breakdown: cost,
      },
    };
  }

  // Investor offer (optional)
  const offerResult = applyOfferIfAny(workingState, plan, rng);
  const culturePenaltyNow = offerResult.penaltyNow || 0;
  const culturePenaltyNext = offerResult.penaltyNext || 0;

  // Spend upfront
  let cashReserve = state.cashReserve - cost.total + offerResult.cashNow;

  // Drain sim for each pillar
  const drain = {};
  const cap = {};
  const meltdown = {};
  const meltdownRatio = {};

  for (const p of PILLAR_LIST) {
    const d = computeDrainForPillar(workingState, plan, p);
    const c = computeCapacityForPillar(workingState, plan, p);
    const m = classifyMeltdown(d, c);

    drain[p] = d;
    cap[p] = c;
    meltdown[p] = m.level;
    meltdownRatio[p] = m.ratio;
  }

  // Apply contingency patches (your “extra cash to keep it running” mechanic)
  const patchRes = applyContingencyPatches(plan, meltdown, cost.contingency);
  const patchedMeltdown = patchRes.patchedMeltdowns;

  // Culture grade (post-offer penalty)
  const cultureRes = computeCultureGrade(plan.tiers, culturePenaltyNow);
  const culture = cultureRes.culture;
  const cultureGrade = cultureRes.grade;

  // Segment satisfaction + returning fans
  const segmentResults = [];
  let returningTotal = 0;

  for (const seg of workingState.segments) {
    const segCount = Math.round(workingState.attendees * seg.sizePct);
    const sat = computeSegmentSatisfaction(seg, plan.tiers, patchedMeltdown, plan.ticketLevel, culturePenaltyNow);
    const retRate = satisfactionToReturnRate(sat);
    const returning = Math.round(segCount * retRate);

    returningTotal += returning;

    segmentResults.push({
      id: seg.id,
      label: seg.label,
      count: segCount,
      satisfaction: sat,
      returnRate: retRate,
      returning,
    });
  }

  // New fans: word-of-mouth + promoter
  const womRate = computeWordOfMouthRate(culture, plan.tiers[PILLARS.MUSIC]);
  const womNew = Math.round(workingState.attendees * womRate * (PRICE_DEMAND_MULT[plan.ticketLevel] ?? 1.0));

  const promoterMeta = PROMOTER_META[plan.promoter] || PROMOTER_META[PROMOTERS.NONE];
  const promoterNew = Math.round(
    workingState.attendees *
    promoterMeta.pct *
    (0.65 + 0.35 * culture) *
    (PRICE_DEMAND_MULT[plan.ticketLevel] ?? 1.0)
  );

  // Next year’s attendance
  const nextAttendees = Math.max(0, returningTotal + womNew + promoterNew);

  // Festival finances:
  // Revenue arrives AFTER festival runs (this is why you can only spend saved profits up front).
  const ticketPrice = TICKET_PRICE[plan.ticketLevel] || 250;
  const revenue = workingState.attendees * ticketPrice;

  const overhead = (workingState.overhead?.fixed ?? DEFAULT_OVERHEAD.fixed) +
                   (workingState.overhead?.perHead ?? DEFAULT_OVERHEAD.perHead) * workingState.attendees;

  // Costs already paid up-front: cost.total
  // Here we also include operational overhead paid from revenue.
  const profitBeforeLoan = revenue - overhead; // upfront spend already removed from cashReserve
  // BUT: we *do* want to reflect the true profit after *all* costs:
  const trueProfitBeforeLoan = revenue - overhead - cost.total + offerResult.cashNow;

  // Loan payment if possible (player can fail)
  let loanRemaining = state.loanRemaining;
  let loanPaid = 0;
  let missedPayment = false;

  if (loanRemaining > 0) {
    const payment = state.loanAnnualPayment;
    if (trueProfitBeforeLoan >= payment) {
      loanPaid = payment;
      loanRemaining = Math.max(0, loanRemaining - payment);
      cashReserve += (trueProfitBeforeLoan - payment);
    } else {
      // Can't pay: keep cash reserve with whatever profit exists; apply penalty next year via report.
      missedPayment = true;
      cashReserve += trueProfitBeforeLoan;
    }
  } else {
    cashReserve += trueProfitBeforeLoan;
  }

  const profitAfterLoan = trueProfitBeforeLoan - loanPaid;

  // Farm status
  const farmPaidOff = loanRemaining <= 0;
  const finished = state.year >= state.loanYears; // after year 10, run ends (newgame+ can restart)

  // Build report (for comedy/story)
  const report = buildYearReport({
    year: state.year,
    attendees: workingState.attendees,
    ticketLevel: plan.ticketLevel,
    ticketPrice,
    promoter: plan.promoter,
    cost,
    offerTaken: offerResult.offerTaken,
    culture,
    cultureGrade,
    drain,
    cap,
    meltdownRatio,
    meltdown: patchedMeltdown,
    segmentResults,
    returningTotal,
    womNew,
    promoterNew,
    nextAttendees,
    revenue,
    overhead,
    profitBeforeLoan: trueProfitBeforeLoan,
    loanPaid,
    loanRemaining,
    profitAfterLoan,
    cashReserveEnd: cashReserve,
    missedPayment,
    culturePenaltyNext,
  });

  // Next state
  const nextState = {
    ...state,
    year: state.year + 1,
    attendees: nextAttendees,
    cashReserve: Math.max(0, Math.round(cashReserve)),
    lastYearProfit: Math.round(profitAfterLoan),
    loanRemaining: Math.round(loanRemaining),
    farmPaidOff,
    finished,
    history: state.history.concat(report.summary),
    // Carry penalties as a “hidden state” in history summary for UI to apply next year.
    // (You can store it elsewhere if you want.)
  };

  return { nextState, report };
}

// -----------------------------
// Report builder (story-first, not hint-first)
// -----------------------------
function buildYearReport(input) {
  const {
    year, attendees, ticketLevel, ticketPrice, promoter, cost, offerTaken,
    culture, cultureGrade, meltdown, meltdownRatio, segmentResults,
    returningTotal, womNew, promoterNew, nextAttendees,
    revenue, overhead, profitBeforeLoan, loanPaid, loanRemaining,
    profitAfterLoan, cashReserveEnd, missedPayment, culturePenaltyNext,
  } = input;

  // Spotlight pillar: highest tier quality (lowest tier number) and worst meltdown.
  const bestPillar = pickBestPillarByTier(input);
  const worstMeltdownPillar = pickWorstMeltdown(meltdown, meltdownRatio);

  const lines = [];

  lines.push(`Year ${year}: ${attendees} fans entered the gates at $${ticketPrice} (${ticketLevel}).`);
  if (promoter !== PROMOTERS.NONE) {
    lines.push(`Promoter hired: ${PROMOTER_META[promoter]?.name || promoter} (paid $${cost.promoterCost}).`);
  } else {
    lines.push(`No promoter this year — you let the vibes speak for themselves.`);
  }

  if (offerTaken) {
    lines.push(`Investor offer taken: ${offerTaken.title} — cash now, soul later 😬.`);
  }

  lines.push(`Culture Grade: ${cultureGrade} (${Math.round(culture * 100)}%).`);

  if (bestPillar) {
    lines.push(`Win highlight: ${pillarsToFlavor(bestPillar)} was surprisingly clutch.`);
  }

  if (worstMeltdownPillar) {
    const lvl = meltdown[worstMeltdownPillar];
    if (lvl === 'warning') lines.push(`Uh oh: ${pillarsToFlavor(worstMeltdownPillar)} got a little spicy (warning).`);
    if (lvl === 'bad') lines.push(`Problem: ${pillarsToFlavor(worstMeltdownPillar)} got cooked (bad).`);
    if (lvl === 'disaster') lines.push(`DISASTER: ${pillarsToFlavor(worstMeltdownPillar)} entered goblin mode (disaster).`);
  } else {
    lines.push(`No system meltdowns — the festival survived with dignity 🫡.`);
  }

  // Segment vibe blurbs (short + comedic)
  const segTop = segmentResults.slice().sort((a, b) => b.satisfaction - a.satisfaction)[0];
  const segBottom = segmentResults.slice().sort((a, b) => a.satisfaction - b.satisfaction)[0];

  if (segTop) lines.push(`${segTop.label} were vibing (satisfaction ${Math.round(segTop.satisfaction * 100)}%).`);
  if (segBottom) lines.push(`${segBottom.label} were side-eyeing you (satisfaction ${Math.round(segBottom.satisfaction * 100)}%).`);

  lines.push(`Next year forecast: ${returningTotal} returning, +${womNew} word-of-mouth, +${promoterNew} promo = ${nextAttendees} total.`);

  lines.push(`Money: revenue (tickets) $${revenue} — overhead $${Math.round(overhead)} — plan spend $${cost.total} → profit before loan $${Math.round(profitBeforeLoan)}.`);
  if (loanPaid > 0) {
    lines.push(`Farm loan payment made: $${loanPaid}. Loan remaining: $${loanRemaining}.`);
  } else if (missedPayment) {
    lines.push(`Farm loan payment MISSED 😵‍💫 (stress penalty next year). Loan remaining: $${loanRemaining}.`);
  } else {
    lines.push(`Farm loan already paid off. You are officially the Dirt Wizard 🌾.`);
  }

  lines.push(`Cash reserve heading into next planning phase: $${cashReserveEnd}.`);

  const summary = {
    year,
    attendees,
    nextAttendees,
    culture: Math.round(culture * 100),
    cultureGrade,
    profitAfterLoan: Math.round(profitAfterLoan),
    cashReserveEnd: Math.round(cashReserveEnd),
    loanRemaining: Math.round(loanRemaining),
    missedPayment,
    carryCulturePenaltyNext: missedPayment ? 0.06 + (culturePenaltyNext || 0) : (culturePenaltyNext || 0),
    meltdowns: { ...meltdown },
  };

  return { lines, summary };
}

function pickBestPillarByTier(input) {
  const tiers = input?.cost?.tiers ? input.cost.tiers : null;
  // We didn't store tiers in cost; so use input via closure? We'll infer from meltdown ratios + culture.
  // Better: spotlight by "non-meltdown pillar with highest quality weight"
  // We'll approximate: pillars with 'ok' meltdown and biggest culture weights get spotlight.
  const weights = CULTURE_WEIGHTS;
  const okPillars = PILLAR_LIST.filter(p => input.meltdown[p] === 'ok');
  if (okPillars.length === 0) return null;
  okPillars.sort((a, b) => (weights[b] || 0) - (weights[a] || 0));
  return okPillars[0];
}

function pickWorstMeltdown(meltdown, ratio) {
  const order = { ok: 0, warning: 1, bad: 2, disaster: 3 };
  let worst = null;
  let worstRank = -1;
  let worstRatio = -1;

  for (const p of PILLAR_LIST) {
    const lvl = meltdown[p] || 'ok';
    const r = ratio[p] || 0;
    const rank = order[lvl] ?? 0;

    if (rank > worstRank || (rank === worstRank && r > worstRatio)) {
      worstRank = rank;
      worstRatio = r;
      worst = lvl === 'ok' ? null : p;
    }
  }

  return worst;
}

function pillarsToFlavor(p) {
  switch (p) {
    case PILLARS.MUSIC: return 'Music Acts';
    case PILLARS.CONCESSIONS: return 'Concessions';
    case PILLARS.FACILITIES: return 'Facilities';
    case PILLARS.LESSONS: return 'Math Lessons';
    case PILLARS.SECURITY: return 'Security';
    default: return p;
  }
}
