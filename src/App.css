import { memo, useEffect, useRef, useState } from 'react'
import './App.css'

/* ==========================================================================
   DADOS
   ========================================================================== */

const NAV = [
  ['inicio', 'Início'],
  ['farmacocinetica', 'Farmacocinética'],
  ['zoom', 'Do tecido à célula'],
  ['farmacodinamica', 'Farmacodinâmica'],
  ['comparar', 'Comparar fármacos'],
  ['quiz', 'Quiz'],
  ['referencias', 'Referências']
]

/* Etapas do percurso do ibuprofeno (farmacocinética) */
const STAGES = [
  {
    key: 'mouth',
    phase: 'Administração',
    title: 'Boca',
    text: 'O comprimido é deglutido com água. Com esta forma farmacêutica não há absorção relevante na boca: o fármaco ainda está íntegro.',
    nursing:
      'Conferir os certos da medicação (paciente, fármaco, dose, via, horário). Comprimidos revestidos não devem ser triturados sem orientação.'
  },
  {
    key: 'esophagus',
    phase: 'Administração',
    title: 'Esôfago',
    text: 'O peristaltismo conduz o comprimido até o estômago. O esôfago não absorve o fármaco.',
    nursing:
      'Oferecer água suficiente e manter o paciente sentado ou em pé: comprimidos retidos podem irritar a mucosa esofágica.'
  },
  {
    key: 'stomach',
    phase: 'Absorção (início)',
    title: 'Estômago',
    text: 'O comprimido começa a se desintegrar e o fármaco se dissolve. O ibuprofeno é um ácido fraco (pKa em torno de 4,4) e pouco solúvel em meio muito ácido; pode haver alguma absorção gástrica, mas ela é limitada.',
    nursing:
      'Se houver desconforto gástrico, administrar com alimento pode ajudar (podendo retardar a velocidade de absorção). Investigar história de úlcera e dor epigástrica.'
  },
  {
    key: 'intestine',
    phase: 'Absorção',
    title: 'Intestino delgado',
    text: 'Principal local de absorção do ibuprofeno. A grande área de superfície (vilosidades e microvilosidades) e o pH menos ácido favorecem a dissolução e a passagem pelo epitélio intestinal. A absorção é rápida e quase completa.',
    nursing:
      'O efeito analgésico costuma surgir em cerca de 30 a 60 minutos, variando conforme a formulação e o paciente.'
  },
  {
    key: 'portal',
    phase: 'Absorção → Metabolismo',
    title: 'Veia porta',
    text: 'Depois de atravessar o epitélio, o fármaco entra nos capilares e segue pela veia porta até o fígado, antes de chegar à circulação geral. É por isso que fármacos orais podem sofrer metabolismo de primeira passagem; no caso do ibuprofeno, a biodisponibilidade oral é alta.',
    nursing: null
  },
  {
    key: 'liver',
    phase: 'Metabolismo',
    title: 'Fígado',
    text: 'O ibuprofeno é metabolizado principalmente por oxidação (CYP2C9) e por glicuronidação, gerando metabólitos inativos e mais hidrossolúveis, que podem ser eliminados pelos rins.',
    nursing:
      'Atenção a pacientes com doença hepática e ao uso concomitante de outros fármacos que interfiram no metabolismo hepático.'
  },
  {
    key: 'blood',
    phase: 'Distribuição',
    title: 'Circulação sistêmica',
    text: 'Pelas veias hepáticas e pela veia cava inferior, o sangue chega ao coração, passa pelos pulmões (circulação pulmonar) e volta ao coração, de onde a aorta o distribui por artérias que se ramificam por todo o corpo. Cerca de 99% do ibuprofeno circula ligado à albumina; apenas a fração livre difunde aos tecidos e interage com os alvos farmacológicos.',
    nursing:
      'Na hipoalbuminemia a fração livre aumenta, o que eleva a relevância clínica de interações e de efeitos adversos.'
  },
  {
    key: 'tissue',
    phase: 'Distribuição',
    title: 'Tecido inflamado',
    text: 'Pela aorta e pelas artérias ilíaca e femoral, o fármaco chega à região inflamada. Ali, a vasodilatação e o aumento da permeabilidade vascular facilitam a saída da fração livre do capilar para o interstício. O fármaco chega às células da região, e o ibuprofeno também pode alcançar o líquido sinovial.',
    nursing:
      'Reavaliar dor, febre e sinais inflamatórios (calor, rubor, edema) e registrar a resposta ao tratamento.'
  },
  {
    key: 'kidney',
    phase: 'Eliminação',
    title: 'Rins',
    text: 'Depois de passar pelos tecidos, o sangue retorna pelas veias ao coração, atravessa de novo os pulmões e é redistribuído pela aorta; a cada passagem, parte do fármaco chega aos rins pelas artérias renais. Os metabólitos (e uma pequena fração de fármaco inalterado) são eliminados sobretudo pela urina, em geral nas primeiras 24 horas. A meia-vida do ibuprofeno é de cerca de 2 horas.',
    nursing:
      'Os AINEs reduzem prostaglandinas renais vasodilatadoras: atenção a idosos, desidratação, doença renal e uso de diuréticos ou IECA. Monitorar diurese e edema.'
  }
]

/* Duração (ms) de cada trecho do percurso */
const DUR = [900, 2200, 2600, 3400, 1800, 1800, 5600, 2200, 6600]

/* ---------- Árvore vascular (gerada de forma determinística) ---------- */
const mirrorPts = (pts) => pts.map(([x, y]) => [420 - x, y])
const fx = (n) => Math.round(n * 10) / 10

function makeRng(seed) {
  let a = seed >>> 0
  return () => {
    a = (a + 0x6d2b79f5) | 0
    let t = Math.imul(a ^ (a >>> 15), 1 | a)
    t = (t + Math.imul(t ^ (t >>> 7), 61 | t)) ^ t
    return ((t ^ (t >>> 14)) >>> 0) / 4294967296
  }
}

/* Spline de Catmull-Rom convertida em curvas de Bézier */
function spline(pts, tension = 1) {
  let d = `M${fx(pts[0][0])} ${fx(pts[0][1])}`
  const k = tension / 6
  for (let i = 0; i < pts.length - 1; i++) {
    const p0 = pts[i - 1] || pts[i]
    const p1 = pts[i]
    const p2 = pts[i + 1]
    const p3 = pts[i + 2] || p2
    d += `C${fx(p1[0] + (p2[0] - p0[0]) * k)} ${fx(p1[1] + (p2[1] - p0[1]) * k)} ${fx(p2[0] - (p3[0] - p1[0]) * k)} ${fx(p2[1] - (p3[1] - p1[1]) * k)} ${fx(p2[0])} ${fx(p2[1])}`
  }
  return d
}

function sampleCurve(pts, steps = 10) {
  const out = []
  for (let i = 0; i < pts.length - 1; i++) {
    const p0 = pts[i - 1] || pts[i]
    const p1 = pts[i]
    const p2 = pts[i + 1]
    const p3 = pts[i + 2] || p2
    const c1 = [p1[0] + (p2[0] - p0[0]) / 6, p1[1] + (p2[1] - p0[1]) / 6]
    const c2 = [p2[0] - (p3[0] - p1[0]) / 6, p2[1] - (p3[1] - p1[1]) / 6]
    for (let s = 0; s < steps; s++) {
      const t = s / steps
      const u = 1 - t
      out.push([
        u * u * u * p1[0] + 3 * u * u * t * c1[0] + 3 * u * t * t * c2[0] + t * t * t * p2[0],
        u * u * u * p1[1] + 3 * u * u * t * c1[1] + 3 * u * t * t * c2[1] + t * t * t * p2[1]
      ])
    }
  }
  out.push(pts[pts.length - 1])
  return out
}

const polyLen = (dn) => {
  let L = 0
  for (let i = 1; i < dn.length; i++) L += Math.hypot(dn[i][0] - dn[i - 1][0], dn[i][1] - dn[i - 1][1])
  return L
}

/* Tubo com afilamento: polígono contínuo da largura w0 até w1 */
function tube(pts, w0, w1) {
  const dn = sampleCurve(pts, 6)
  const n = dn.length
  const A = []
  const B = []
  for (let i = 0; i < n; i++) {
    const a = dn[Math.max(0, i - 1)]
    const b = dn[Math.min(n - 1, i + 1)]
    let tx = b[0] - a[0]
    let ty = b[1] - a[1]
    const tl = Math.hypot(tx, ty) || 1
    tx /= tl
    ty /= tl
    const w = (w0 + (w1 - w0) * (i / (n - 1))) / 2
    A.push([dn[i][0] - ty * w, dn[i][1] + tx * w])
    B.push([dn[i][0] + ty * w, dn[i][1] - tx * w])
  }
  return 'M' + A.concat(B.reverse()).map((p) => `${fx(p[0])} ${fx(p[1])}`).join('L') + 'Z'
}

/* Regiões do corpo, para os ramos não saírem da silhueta */
const inPoly = (x, y, poly) => {
  let c = false
  for (let i = 0, j = poly.length - 1; i < poly.length; j = i++) {
    const [xi, yi] = poly[i]
    const [xj, yj] = poly[j]
    if (yi > y !== yj > y && x < ((xj - xi) * (y - yi)) / (yj - yi) + xi) c = !c
  }
  return c
}

const ellipsePoly = (cx, cy, rx, ry) =>
  Array.from({ length: 24 }, (_, i) => [
    cx + rx * Math.cos((i / 24) * Math.PI * 2),
    cy + ry * Math.sin((i / 24) * Math.PI * 2)
  ])

const ARM_L = [[122, 158], [104, 166], [98, 196], [92, 232], [72, 322], [68, 340], [84, 346], [104, 352], [128, 262]]
const LEG_L = [[134, 450], [124, 578], [120, 596], [118, 606], [134, 606], [178, 606], [182, 580], [206, 490], [210, 474]]
const LUNG_R = [[200, 160], [178, 150], [150, 160], [141, 198], [143, 235], [160, 250], [186, 254], [200, 244]]
const LIVER = [[138, 262], [150, 254], [176, 253], [198, 258], [207, 266], [206, 276], [198, 290], [178, 300], [156, 303], [136, 296], [130, 282]]

const BODY_POLYS = [
  [[190, 128], [150, 132], [122, 152], [118, 170], [116, 330], [118, 400], [125, 450], [134, 474], [286, 474], [295, 450], [302, 400], [304, 330], [302, 170], [298, 152], [270, 132], [230, 128]],
  ellipsePoly(210, 72, 40, 45),
  [[192, 106], [192, 134], [228, 134], [228, 106]],
  ARM_L,
  mirrorPts(ARM_L),
  LEG_L,
  mirrorPts(LEG_L)
]

const inBody = (x, y) =>
  [[0, 0], [2.4, 0], [-2.4, 0], [0, 2.4], [0, -2.4]].every(([dx, dy]) =>
    BODY_POLYS.some((p) => inPoly(x + dx, y + dy, p))
  )
const inLungR = (x, y) => [[0, 0], [2, 0], [-2, 0], [0, 2], [0, -2]].every(([dx, dy]) => inPoly(x + dx, y + dy, LUNG_R))
const inLungL = (x, y) => inLungR(420 - x, y)
const inLiver = (x, y) => [[0, 0], [2, 0], [-2, 0], [0, 2], [0, -2]].every(([dx, dy]) => inPoly(x + dx, y + dy, LIVER))
const inGut = (x, y) => x > 152 && x < 272 && y > 360 && y < 444 && inBody(x, y)

/* Ramificação procedural */
function branches(out, pts, o, depth, w, len, rnd) {
  const dn = sampleCurve(pts, 10)
  const L = polyLen(dn)
  const gap = o.gap * Math.pow(0.82, o.depth - depth)
  const n = Math.max(1, Math.round(L / gap))
  for (let k = 0; k < n; k++) {
    const u = o.from + (o.to - o.from) * ((k + 0.2 + rnd() * 0.6) / n)
    const idx = Math.max(0, Math.min(dn.length - 2, Math.floor(u * (dn.length - 1))))
    const p = dn[idx]
    const q = dn[idx + 1]
    let tx = q[0] - p[0]
    let ty = q[1] - p[1]
    const tl = Math.hypot(tx, ty) || 1
    tx /= tl
    ty /= tl
    const side = o.side || (k % 2 ? 1 : -1) * (rnd() < 0.8 ? 1 : -1)
    const ang = side * (o.ang[0] + rnd() * (o.ang[1] - o.ang[0]))
    const c = Math.cos(ang)
    const s = Math.sin(ang)
    const dir0 = [tx * c - ty * s, tx * s + ty * c]
    const bend = (rnd() - 0.5) * 0.7
    let length = len[0] + rnd() * (len[1] - len[0])
    let pts2 = null
    for (let attempt = 0; attempt < 3 && !pts2; attempt++) {
      const segs = 4
      const arr = [[p[0], p[1]]]
      let [dx, dy] = dir0
      let x = p[0]
      let y = p[1]
      let ok = true
      for (let i = 0; i < segs; i++) {
        const a = bend / segs
        const ca = Math.cos(a)
        const sa = Math.sin(a)
        const nx = dx * ca - dy * sa
        const ny = dx * sa + dy * ca
        dx = nx * (1 - o.follow) + tx * o.follow
        dy = ny * (1 - o.follow) + ty * o.follow
        const dl = Math.hypot(dx, dy) || 1
        dx /= dl
        dy /= dl
        x += (dx * length) / segs
        y += (dy * length) / segs
        if (!o.region(x, y)) {
          ok = false
          break
        }
        arr.push([x, y])
      }
      if (ok) pts2 = arr
      else length *= 0.62
      if (length < 4) break
    }
    if (!pts2) continue
    if (depth === o.depth && w >= 0.9) {
      out.push({ t: 2, kind: o.kind, d: tube(pts2, w * 1.25, Math.max(0.5, w * 0.45)), a: 0.95 })
    } else {
      out.push({ t: 0, kind: o.kind, d: spline(pts2), w: Math.max(0.5, w), a: Math.max(0.55, 1 - (o.depth - depth) * 0.14) })
    }
    if (depth > 1) branches(out, pts2, o, depth - 1, w * o.wd, [len[0] * o.ld, len[1] * o.ld], rnd)
  }
}

/* Fios colaterais: nascem do vaso principal, correm paralelos e voltam a se unir a ele */
function collateral(out, pts, o) {
  const dn = sampleCurve(pts, 8)
  const n = dn.length
  const i0 = Math.floor(n * o.from)
  const i1 = Math.floor(n * o.to)
  const res = []
  for (let i = i0; i <= i1; i += 4) {
    const a = dn[Math.max(0, i - 1)]
    const b = dn[Math.min(n - 1, i + 1)]
    let tx = b[0] - a[0]
    let ty = b[1] - a[1]
    const l = Math.hypot(tx, ty) || 1
    tx /= l
    ty /= l
    const k = (i - i0) / Math.max(1, i1 - i0)
    const env = Math.pow(Math.sin(Math.PI * k), 0.7)
    const off = env * (o.off + o.amp * Math.sin(k * o.freq * Math.PI * 2 + o.phase))
    const x = dn[i][0] - ty * off
    const y = dn[i][1] + tx * off
    if (inBody(x, y)) res.push([x, y])
  }
  if (res.length > 3) out.push({ t: 2, kind: o.kind, d: tube(res, o.w0, o.w1), a: o.a || 0.9 })
}

function trunk(out, spec, rnd) {
  const o = Object.assign(
    {
      depth: 2,
      gap: 16,
      ang: [0.35, 0.95],
      len: [14, 26],
      follow: 0.1,
      from: 0.08,
      to: 0.96,
      side: 0,
      ld: 0.6,
      wd: 0.62,
      flow: false,
      region: inBody
    },
    spec
  )
  out.push({
    t: 1,
    kind: o.kind,
    d: tube(o.pts, o.w0, o.w1),
    c: spline(o.pts),
    w: (o.w0 + o.w1) / 2,
    flow: o.flow,
    id: o.id || ''
  })
  if (o.depth > 0) branches(out, o.pts, o, o.depth, Math.max(0.7, o.w1 * 0.55), o.len, rnd)
}

/* ---------- Eixos principais (compartilhados com o percurso do comprimido) ---------- */
const AORTA_TOP = [[223, 209], [225, 198], [224, 184], [226, 168], [234, 158], [243, 162], [248, 174], [248, 192], [247, 212], [244, 234], [236, 254], [226, 268], [221, 286], [220, 308], [221, 330]]
const AORTA_BOTTOM = [[221, 330], [222, 360], [223, 395], [224, 425], [224, 440]]
const HEART_LUNG = [[212, 219], [216, 211], [217, 200], [215, 188], [207, 181], [195, 183], [183, 191], [172, 202], [164, 214], [162, 227], [170, 233], [181, 226], [192, 217], [203, 210], [213, 209]]
const IVC = [[214, 446], [210, 424], [207, 396], [206, 366], [206, 336], [205, 306], [204, 278], [205, 258], [207, 242], [210, 229], [212, 219]]
const SVC = [[206, 150], [207, 168], [209, 188], [211, 204], [212, 216]]
const LEG_A_R = [[224, 440], [233, 451], [242, 467], [248, 488], [254, 506], [258, 520], [262, 536], [265, 556], [268, 578], [270, 598]]
const LEG_A_L = [[224, 440], [210, 451], [196, 464], [183, 478], [172, 494], [166, 510], [163, 528], [159, 552], [154, 576], [151, 598]]
const LEG_V_R = [[266, 598], [262, 578], [258, 556], [253, 536], [249, 520], [244, 504], [238, 488], [231, 470], [224, 456], [216, 448]]
const LEG_V_L = [[154, 598], [158, 578], [161, 556], [166, 536], [169, 520], [175, 504], [183, 488], [193, 470], [203, 456], [212, 448]]
const RENAL_A_R = [[221, 331], [236, 333], [250, 331], [262, 327]]
const RENAL_A_L = [[221, 342], [205, 347], [188, 347], [172, 344], [160, 340]]
const PORTAL = [[212, 402], [200, 392], [192, 372], [188, 350], [186, 326], [184, 302]]
const LIVER_PATH = [[184, 302], [172, 288], [176, 274], [190, 267], [205, 261]]
const IVC_UP = [[205, 261], [205, 258], [207, 242], [210, 229], [212, 219]]

const joinPts = (...segs) => {
  const out = []
  segs.forEach((s) =>
    s.forEach((p) => {
      const l = out[out.length - 1]
      if (!l || Math.hypot(l[0] - p[0], l[1] - p[1]) > 0.5) out.push(p)
    })
  )
  return out
}

function buildVascular() {
  const out = []
  const rnd = makeRng(20261112)

  const armA = [[212, 150], [198, 152], [178, 152], [156, 156], [138, 163], [127, 174], [117, 192], [108, 212], [102, 236], [98, 262], [94, 290], [90, 316], [88, 338]]
  const armAL = [[242, 160], [260, 153], [280, 153], [294, 162], [303, 176], [309, 198], [314, 222], [319, 246], [323, 272], [327, 298], [331, 322], [333, 342]]
  const armV = [[93, 334], [96, 308], [100, 282], [104, 256], [109, 232], [114, 210], [121, 192], [131, 178], [145, 169], [166, 163], [190, 159], [204, 155], [207, 152]]
  const armVL = [[329, 334], [323, 308], [319, 282], [315, 256], [311, 232], [306, 210], [300, 192], [290, 177], [276, 164], [258, 154], [240, 148], [222, 147], [209, 150]]
  const cephR = [[90, 331], [93, 302], [98, 272], [104, 242], [111, 216], [121, 196], [134, 181], [146, 171]]
  const cephL = mirrorPts(cephR)

  const LIMB = { depth: 2, gap: 30, ang: [0.12, 0.38], len: [26, 46], follow: 0.7, from: 0.06, to: 0.9, ld: 0.5, wd: 0.6 }
  const HEAD = { depth: 2, gap: 16, ang: [0.35, 0.8], len: [12, 22], follow: 0.45, from: 0.3, to: 0.95, ld: 0.6, wd: 0.6 }
  const LUNG = { depth: 2, gap: 15, ang: [0.45, 0.9], len: [14, 26], follow: 0.2, ld: 0.65, wd: 0.6 }
  const GUT = { depth: 2, gap: 12, ang: [0.5, 1.0], len: [16, 28], follow: 0.1, ld: 0.6, wd: 0.6, region: inGut }

  /* ---------- Veias (periferia → coração) ---------- */
  trunk(out, { kind: 'v', id: 'ivc', pts: IVC, w0: 3.6, w1: 5, depth: 0, flow: true }, rnd)
  trunk(out, { kind: 'v', id: 'svc', pts: SVC, w0: 3.4, w1: 3.6, depth: 0, flow: true }, rnd)
  trunk(out, { ...HEAD, kind: 'v', pts: [[190, 56], [191, 76], [194, 96], [197, 112], [200, 126], [203, 138], [206, 150]], w0: 1.4, w1: 2.8, flow: true }, rnd)
  trunk(out, { ...HEAD, kind: 'v', pts: [[230, 56], [229, 76], [227, 96], [224, 112], [221, 126], [217, 138], [212, 148], [208, 152]], w0: 1.4, w1: 2.8, flow: true }, rnd)
  trunk(out, { ...LIMB, kind: 'v', pts: armV, w0: 1.4, w1: 2.8, flow: true }, rnd)
  trunk(out, { ...LIMB, kind: 'v', pts: armVL, w0: 1.4, w1: 2.8, flow: true }, rnd)
  trunk(out, { ...LIMB, kind: 'v', pts: cephR, w0: 1, w1: 1.7, gap: 26 }, rnd)
  trunk(out, { ...LIMB, kind: 'v', pts: cephL, w0: 1, w1: 1.7, gap: 26 }, rnd)
  trunk(out, { ...LIMB, kind: 'v', pts: LEG_V_R, w0: 1.6, w1: 3.2, flow: true }, rnd)
  trunk(out, { ...LIMB, kind: 'v', pts: LEG_V_L, w0: 1.6, w1: 3.2, flow: true }, rnd)
  trunk(out, { ...LIMB, kind: 'v', pts: [[272, 596], [275, 572], [277, 548], [277, 524], [273, 502], [266, 484], [257, 470], [246, 462]], w0: 1, w1: 1.9, gap: 26 }, rnd)
  trunk(out, { ...LIMB, kind: 'v', pts: [[148, 596], [145, 572], [143, 548], [143, 524], [147, 502], [154, 484], [163, 470], [174, 462]], w0: 1, w1: 1.9, gap: 26 }, rnd)
  trunk(out, { kind: 'v', pts: [[263, 321], [247, 321], [231, 326], [215, 331], [206, 335]], w0: 1.6, w1: 2.4, depth: 0, flow: true }, rnd)
  trunk(out, { kind: 'v', pts: [[161, 334], [176, 333], [192, 334], [206, 336]], w0: 1.6, w1: 2.4, depth: 0, flow: true }, rnd)

  /* ---------- Artérias (coração → periferia) ---------- */
  trunk(out, { kind: 'a', id: 'aorta', pts: joinPts(AORTA_TOP, AORTA_BOTTOM), w0: 5.2, w1: 3.4, depth: 0, flow: true }, rnd)
  trunk(out, { kind: 'a', pts: [[226, 168], [219, 158], [212, 150]], w0: 3.2, w1: 3, depth: 0 }, rnd)
  trunk(out, { ...HEAD, kind: 'a', pts: [[212, 150], [208, 138], [205, 126], [203, 110], [201, 96], [199, 80], [197, 62], [199, 46]], w0: 2.8, w1: 1.1, flow: true }, rnd)
  trunk(out, { ...HEAD, kind: 'a', pts: [[234, 158], [226, 150], [221, 138], [219, 124], [220, 108], [222, 92], [224, 74], [222, 56], [220, 44]], w0: 2.8, w1: 1.1, flow: true }, rnd)
  trunk(out, { ...LIMB, kind: 'a', pts: armA, w0: 2.8, w1: 1.1, flow: true }, rnd)
  trunk(out, { ...LIMB, kind: 'a', pts: armAL, w0: 2.8, w1: 1.1, flow: true }, rnd)
  trunk(out, { ...LIMB, kind: 'a', pts: LEG_A_R, w0: 3.4, w1: 1.3, flow: true }, rnd)
  trunk(out, { ...LIMB, kind: 'a', pts: LEG_A_L, w0: 3.4, w1: 1.3, flow: true }, rnd)
  trunk(out, { kind: 'a', pts: RENAL_A_R, w0: 2.2, w1: 1.6, depth: 1, gap: 14, ang: [0.5, 0.9], len: [6, 10] }, rnd)
  trunk(out, { kind: 'a', pts: RENAL_A_L, w0: 2.2, w1: 1.6, depth: 1, gap: 14, ang: [0.5, 0.9], len: [6, 10] }, rnd)
  trunk(out, { ...GUT, kind: 'a', pts: [[222, 372], [215, 384], [207, 396], [197, 404]], w0: 2, w1: 1.2 }, rnd)
  trunk(out, { ...GUT, kind: 'a', pts: [[223, 408], [230, 414], [240, 420], [252, 424]], w0: 1.8, w1: 1.2 }, rnd)

  /* ---------- Circulação pulmonar ---------- */
  const paR = [[216, 205], [217, 194], [214, 185], [206, 181], [195, 184], [183, 192], [172, 203], [164, 214], [162, 227]]
  const pvR = [[162, 227], [170, 233], [181, 226], [192, 217], [203, 210], [213, 209]]
  trunk(out, { ...LUNG, kind: 'v', pts: paR, w0: 3, w1: 1.8, region: inLungR, from: 0.3 }, rnd)
  trunk(out, { ...LUNG, kind: 'a', pts: pvR, w0: 1.8, w1: 2.8, region: inLungR, from: 0.05, to: 0.8 }, rnd)
  const paL = [[216, 204], [218, 190], [227, 183], [240, 186], [251, 194], [259, 206], [264, 221]]
  const pvL = [[264, 221], [262, 233], [254, 226], [246, 218], [236, 212], [224, 210]]
  trunk(out, { ...LUNG, kind: 'v', pts: paL, w0: 3, w1: 1.8, region: inLungL, from: 0.4 }, rnd)
  trunk(out, { ...LUNG, kind: 'a', pts: pvL, w0: 1.8, w1: 2.8, region: inLungL, from: 0.05, to: 0.8 }, rnd)

  /* ---------- Veia porta e fígado ---------- */
  trunk(out, { ...GUT, kind: 'p', id: 'portal', pts: PORTAL, w0: 2.4, w1: 3.2, from: 0, to: 0.55, flow: true }, rnd)
  trunk(out, { kind: 'p', pts: [[184, 302], [176, 292], [168, 284]], w0: 3, w1: 2.2, depth: 2, gap: 9, ang: [0.4, 0.9], len: [10, 18], follow: 0.2, region: inLiver }, rnd)
  trunk(out, { kind: 'v', pts: [[150, 272], [162, 268], [176, 266], [190, 264], [205, 261]], w0: 1.6, w1: 2.6, depth: 2, gap: 10, ang: [0.45, 0.9], len: [10, 18], follow: 0.2, region: inLiver, to: 0.7 }, rnd)

  /* ---------- Fios colaterais nos membros (aspecto de rede) ---------- */
  const strands = (pts, kindA, kindB, amp) => {
    collateral(out, pts, { kind: kindA, from: 0.1, to: 0.95, off: 3, amp, freq: 1.5, phase: 0.4, w0: 1.1, w1: 0.55 })
    collateral(out, pts, { kind: kindB, from: 0.18, to: 0.98, off: -3, amp, freq: 1.2, phase: 2.2, w0: 1.1, w1: 0.55 })
    collateral(out, pts, { kind: kindA, from: 0.35, to: 0.95, off: 5, amp: amp * 0.8, freq: 2, phase: 4, w0: 0.9, w1: 0.5, a: 0.75 })
    collateral(out, pts, { kind: kindB, from: 0.05, to: 0.7, off: -5, amp: amp * 0.8, freq: 1.8, phase: 1, w0: 0.9, w1: 0.5, a: 0.75 })
  }
  strands(armA, 'a', 'v', 4)
  strands(armAL, 'a', 'v', 4)
  strands(LEG_A_R, 'a', 'v', 6)
  strands(LEG_A_L, 'a', 'v', 6)

  /* ---------- Leito capilar do tecido inflamado (joelho) ---------- */
  const kneeRegion = (x, y) => inBody(x, y) && Math.hypot(x - 258, y - 520) < 30
  for (let i = 0; i < 8; i++) {
    const a = (i / 8) * Math.PI * 2 + 0.3
    const pts = [
      [258 + Math.cos(a) * 5, 520 + Math.sin(a) * 5],
      [258 + Math.cos(a + 0.15) * 12, 520 + Math.sin(a + 0.15) * 12],
      [258 + Math.cos(a + 0.45) * 20, 520 + Math.sin(a + 0.45) * 20]
    ]
    if (pts.every(([x, y]) => kneeRegion(x, y))) {
      const kind = i % 2 ? 'v' : 'a'
      out.push({ t: 0, kind, d: spline(pts), w: 0.8, a: 0.9 })
      branches(out, pts, { kind, depth: 1, gap: 9, ang: [0.6, 1.0], follow: 0.3, from: 0.4, to: 1, side: 0, ld: 0.6, wd: 0.7, region: kneeRegion }, 1, 0.55, [5, 9], rnd)
    }
  }

  return out
}

const VESSELS = buildVascular()

/* ---------- Percurso do comprimido pelos vasos ---------- */
const sp = (pts) => spline(pts, 0.9)
const ROUTE = [
  'M210 92 L210 112',
  'M210 112 L212 124 L214 134 L214 252 Q216 266 232 269',
  'M232 269 C246 272 258 282 262 294 L266 308 C264 320 252 322 240 318 L228 314',
  'M228 314 L216 314 C198 318 188 334 192 350 C196 366 206 374 214 380 C240 378 258 388 254 399 C250 410 228 406 212 402',
  sp(PORTAL),
  sp(LIVER_PATH),
  sp(joinPts(IVC_UP, HEART_LUNG, AORTA_TOP, AORTA_BOTTOM)),
  sp(joinPts(LEG_A_R.slice(0, 6), [[255, 521]])),
  sp(joinPts([[255, 521]], LEG_V_R.slice(4), IVC, HEART_LUNG, AORTA_TOP.slice(0, 14), RENAL_A_R))
]

const V_TRUNKS = VESSELS.filter((v) => v.t === 1)
const V_THIN = VESSELS.filter((v) => v.t !== 1)

const SMALL_INTESTINE =
  'M214 380 C240 378 258 388 254 399 C250 410 228 406 212 402 C190 398 170 404 172 414 C174 424 200 418 224 422 C244 425 252 434 238 438 C222 442 200 436 182 438 C172 439 166 436 162 434'

const ADME = [
  { k: 'A', name: 'Absorção', on: [2, 3, 4] },
  { k: 'D', name: 'Distribuição', on: [6, 7] },
  { k: 'M', name: 'Metabolismo', on: [5] },
  { k: 'E', name: 'Excreção', on: [8] }
]

const ORGANS = {
  esophagus: {
    name: 'Esôfago',
    description:
      'Tubo muscular que conduz o comprimido da faringe ao estômago por peristaltismo. Não absorve o fármaco; orientar a ingestão com líquido reduz o risco de lesão por contato.'
  },
  stomach: {
    name: 'Estômago',
    description:
      'Recebe o comprimido após a deglutição. O ibuprofeno se dissolve no trato gastrointestinal e é absorvido predominantemente no intestino delgado. A inibição de COX-1 reduz prostaglandinas que protegem a mucosa gástrica.'
  },
  intestine: {
    name: 'Intestino delgado',
    description:
      'É o principal local de absorção do ibuprofeno. Após atravessar o epitélio intestinal, o fármaco alcança a circulação portal.'
  },
  liver: {
    name: 'Fígado',
    description:
      'Recebe o sangue do intestino pela circulação portal. Participa do metabolismo do ibuprofeno, principalmente por oxidação (CYP2C9) e conjugação (glicuronidação).'
  },
  blood: {
    name: 'Circulação sanguínea',
    description:
      'Transporta o ibuprofeno pelo organismo: as artérias (em vermelho) levam o sangue do coração aos tecidos e as veias (em azul) o devolvem. No plasma, grande parte do fármaco circula ligada à albumina; só a fração livre se distribui aos tecidos.'
  },
  kidney: {
    name: 'Rins',
    description:
      'Principal via de eliminação dos metabólitos. As prostaglandinas renais ajudam a manter o fluxo sanguíneo em algumas situações; por isso os AINEs exigem cautela em idosos, desidratação e doença renal.'
  }
}

const ADME_CARDS = [
  {
    letter: 'A',
    title: 'Absorção',
    text: 'Rápida e quase completa por via oral; principal local: intestino delgado. Alimentos podem retardar a velocidade, com pouca alteração na extensão da absorção.'
  },
  {
    letter: 'D',
    title: 'Distribuição',
    text: 'Cerca de 99% ligado à albumina. Volume de distribuição baixo (aprox. 0,1 a 0,2 L/kg). Atinge tecidos inflamados e o líquido sinovial.'
  },
  {
    letter: 'M',
    title: 'Metabolismo',
    text: 'Hepático: oxidação (principalmente CYP2C9) e glicuronidação. O isômero R sofre inversão quiral parcial para o isômero S, que é o mais ativo sobre a COX.'
  },
  {
    letter: 'E',
    title: 'Excreção',
    text: 'Renal, quase toda como metabólitos, em geral nas primeiras 24 horas. Meia-vida de aproximadamente 2 horas.'
  }
]

const ZOOM = [
  {
    tab: '1 · Tecido',
    title: 'Região inflamada',
    text: 'Histamina, bradicinina e prostaglandinas provocam vasodilatação e aumento da permeabilidade vascular, o que explica calor, rubor e edema. A fração livre do fármaco deixa o capilar e se difunde pelo interstício até as células.'
  },
  {
    tab: '2 · Célula',
    title: 'Dentro da célula',
    text: 'O ibuprofeno é lipofílico e atravessa a membrana principalmente por difusão passiva, alcançando o citoplasma. É ali, junto às membranas, que estão as enzimas ciclo-oxigenases (COX).'
  },
  {
    tab: '3 · Membrana e enzimas',
    title: 'Membrana e alvos moleculares',
    text: 'Os fosfolipídios da bicamada são a fonte do ácido araquidônico. A partir daqui começa a farmacodinâmica: a liberação do substrato, a ação das COX e o bloqueio exercido pelo ibuprofeno.'
  }
]

const PD_STEPS = [
  {
    title: 'Estímulo inflamatório',
    text: 'Lesão tecidual, microrganismos e citocinas ativam as células da região. Sinais intracelulares, como o aumento de Ca²⁺ no citosol, preparam a ativação de enzimas ligadas à membrana.',
    drug: 'O ibuprofeno não bloqueia o estímulo inicial nem a sinalização que leva à ativação celular.'
  },
  {
    title: 'Ativação da fosfolipase A₂',
    text: 'O aumento de Ca²⁺ e a fosforilação ativam a fosfolipase A₂ citosólica (cPLA₂), que se associa à membrana e hidrolisa fosfolipídios, rompendo a ligação éster na posição sn-2.',
    drug: 'A PLA₂ não é o alvo do ibuprofeno. Os corticosteroides, por outro lado, reduzem a liberação de ácido araquidônico em parte por meio de proteínas como a anexina A1.'
  },
  {
    title: 'Liberação do ácido araquidônico',
    text: 'O ácido araquidônico, um ácido graxo de 20 carbonos, é liberado dos fosfolipídios de membrana e fica disponível no citoplasma como substrato das ciclo-oxigenases.',
    drug: 'O ácido araquidônico continua sendo liberado. O ibuprofeno age na etapa seguinte, impedindo que a enzima o utilize.'
  },
  {
    title: 'Ação das ciclo-oxigenases',
    text: 'As COX-1 e COX-2 convertem o ácido araquidônico em PGG₂ (atividade ciclo-oxigenase) e depois em PGH₂ (atividade peroxidase). A COX-1 é constitutiva; a COX-2 é induzida por estímulos inflamatórios.',
    drug: 'O ibuprofeno compete com o ácido araquidônico pelo canal hidrofóbico da enzima e se liga de forma reversível, bloqueando a conversão em PGG₂ e PGH₂. Ele inibe COX-1 e COX-2 (não seletivo).'
  },
  {
    title: 'Prostanoides e seus efeitos',
    text: 'A PGH₂ é convertida por sintases específicas de cada tecido em prostanoides (PGE₂, PGI₂, TXA₂, PGD₂ e PGF₂α), que participam de dor, febre, vasodilatação, edema, proteção gástrica e agregação plaquetária.',
    drug: 'Com menos prostanoides, há menor sensibilização de nociceptores (analgesia), menos febre e menos vasodilatação e edema. A inibição da COX-1 explica parte dos efeitos adversos: menor proteção da mucosa gástrica, menor perfusão renal em situações dependentes de prostaglandinas e redução reversível do TXA₂ plaquetário.'
  }
]

const EFFECTS = [
  {
    pg: 'PGE₂',
    role: 'Sensibiliza nociceptores, induz febre (hipotálamo), vasodilatação e edema; também protege a mucosa gástrica e participa da regulação renal.',
    inhib: 'Menos dor, febre e edema (efeito terapêutico). Risco gastrointestinal e renal (efeito adverso).'
  },
  {
    pg: 'PGI₂',
    role: 'Vasodilatação, inibição da agregação plaquetária e citoproteção gástrica; produzida em grande parte via COX-2 no endotélio.',
    inhib: 'Pode contribuir para o risco cardiovascular observado com alguns AINEs.'
  },
  {
    pg: 'TXA₂',
    role: 'Produzido pelas plaquetas via COX-1: promove agregação plaquetária e vasoconstrição.',
    inhib: 'Redução da agregação plaquetária (reversível com o ibuprofeno) e maior tendência a sangramento.'
  }
]

const DRUGS = [
  {
    key: 'ibu',
    name: 'Ibuprofeno',
    cls: 'AINE · derivado do ácido propiônico',
    target: 'COX-1 e COX-2',
    rev: 'Reversível, competitiva',
    half: '≈ 2 h',
    sel: 0.45,
    selText: 'Não seletivo',
    mech: 'Inibe de forma reversível as duas isoformas de COX, reduzindo a síntese de prostaglandinas e de tromboxano. Tem ação analgésica, antipirética e anti-inflamatória.',
    attention: [
      'Gastrointestinal: dispepsia, úlcera e sangramento',
      'Renal: pode reduzir a perfusão e causar retenção de líquidos',
      'Cardiovascular: risco maior com doses altas e uso prolongado',
      'Pode interferir no efeito antiplaquetário do AAS em baixa dose'
    ],
    nursing: [
      'Administrar com água; com alimento se houver desconforto gástrico',
      'Observar fezes escuras, dor epigástrica, edema e redução da diurese',
      'Cautela em idosos, doença renal e uso de anticoagulantes, diuréticos ou IECA',
      'Na gestação exige avaliação médica; em geral é evitado no terceiro trimestre'
    ]
  },
  {
    key: 'dic',
    name: 'Diclofenaco',
    cls: 'AINE · derivado do ácido acético',
    target: 'COX-1 e COX-2, com preferência relativa pela COX-2',
    rev: 'Reversível',
    half: '≈ 1 a 2 h',
    sel: 0.72,
    selText: 'Preferência relativa por COX-2',
    mech: 'Inibe COX-1 e COX-2, com maior preferência pela COX-2 do que o ibuprofeno. É muito usado em dor e inflamação musculoesquelética.',
    attention: [
      'O risco cardiovascular merece atenção em uso prolongado ou em doença cardíaca',
      'Risco gastrointestinal e renal semelhante ao dos demais AINEs',
      'Pode elevar enzimas hepáticas: monitorar em tratamentos prolongados'
    ],
    nursing: [
      'Atenção a doença cardiovascular, renal e hepática',
      'Orientar sobre sinais de sangramento digestivo',
      'Seguir a via e a formulação prescritas: oral, injetável e tópica têm perfis distintos'
    ]
  },
  {
    key: 'nap',
    name: 'Naproxeno',
    cls: 'AINE · derivado do ácido propiônico',
    target: 'COX-1 e COX-2',
    rev: 'Reversível',
    half: '≈ 12 a 17 h',
    sel: 0.42,
    selText: 'Não seletivo',
    mech: 'Inibe de forma reversível COX-1 e COX-2. A meia-vida longa permite intervalos maiores entre as doses.',
    attention: [
      'Risco gastrointestinal relevante, em especial em idosos',
      'Meia-vida longa: efeitos adversos podem persistir após a suspensão',
      'Cuidados renais e cardiovasculares como nos demais AINEs'
    ],
    nursing: [
      'Respeitar o intervalo prescrito: o acúmulo é maior que com fármacos de meia-vida curta',
      'Observar sangramentos, edema e alterações da diurese',
      'Administrar com água e, se necessário, com alimento'
    ]
  },
  {
    key: 'aas',
    name: 'Ácido acetilsalicílico',
    cls: 'AINE · salicilato',
    target: 'COX-1 (preferencial) e COX-2',
    rev: 'Irreversível (acetilação)',
    half: '≈ 15 a 20 min (AAS); o salicilato depende da dose',
    sel: 0.2,
    selText: 'Preferência por COX-1',
    mech: 'Acetila de forma covalente um resíduo de serina da COX. Como a plaqueta não sintetiza novas proteínas, a inibição do TXA₂ plaquetário dura toda a vida da plaqueta (cerca de 7 a 10 dias).',
    attention: [
      'Risco de sangramento por efeito antiplaquetário prolongado',
      'Irritação e lesão gastrointestinal',
      'Síndrome de Reye em crianças e adolescentes com infecção viral',
      'Hipersensibilidade em pacientes com asma sensível a AINEs'
    ],
    nursing: [
      'Verificar a indicação: baixa dose (antiplaquetário) versus doses analgésicas e anti-inflamatórias',
      'Revisar o uso de anticoagulantes e de outros AINEs',
      'Não administrar a crianças e adolescentes com quadro viral sem orientação médica'
    ]
  },
  {
    key: 'cort',
    name: 'Corticosteroides',
    cls: 'Hormônio esteroide · glicocorticoide (ex.: prednisona, dexametasona)',
    target: 'Receptor intracelular de glicocorticoides',
    rev: 'Modulação da expressão gênica (efeito não imediato)',
    half: 'Plasmática curta; efeito biológico mais prolongado (varia por fármaco)',
    sel: null,
    selText: '',
    mech: 'Atravessa a membrana, liga-se ao receptor de glicocorticoides no citoplasma e o complexo modula a transcrição gênica no núcleo: reduz a expressão de COX-2, de citocinas e de outras proteínas pró-inflamatórias, e aumenta proteínas anti-inflamatórias. Não é um inibidor direto da COX como o ibuprofeno.',
    attention: [
      'Hiperglicemia e retenção de líquidos',
      'Imunossupressão e maior risco de infecções',
      'Uso prolongado: osteoporose, alterações de humor e supressão do eixo hipotálamo-hipófise-adrenal',
      'A interrupção abrupta após uso prolongado pode causar insuficiência adrenal'
    ],
    nursing: [
      'Monitorar glicemia, pressão arterial e sinais de infecção',
      'Não suspender bruscamente sem orientação',
      'Administrar conforme a prescrição (em dose única diária, frequentemente pela manhã)'
    ]
  }
]

const GENOMIC = [
  'Entra na célula',
  'Liga-se ao receptor (GR) no citoplasma',
  'O complexo vai ao núcleo',
  'Modula genes: menos COX-2 e citocinas, mais anexina A1'
]

const QUIZ = [
  {
    q: 'Qual é o principal local de absorção do ibuprofeno administrado por via oral?',
    options: ['Estômago', 'Intestino delgado', 'Intestino grosso', 'Esôfago'],
    answer: 1,
    why: 'A grande área de superfície do intestino delgado e o pH menos ácido favorecem a dissolução e a absorção. Pode haver alguma absorção gástrica, mas ela é limitada.'
  },
  {
    q: 'A farmacodinâmica estuda:',
    options: [
      'O que o organismo faz com o fármaco',
      'O que o fármaco faz no organismo',
      'Apenas a eliminação renal',
      'Somente os efeitos adversos'
    ],
    answer: 1,
    why: 'Farmacocinética é o que o organismo faz com o fármaco (ADME); farmacodinâmica é o que o fármaco faz no organismo (alvos, vias bioquímicas e efeitos).'
  },
  {
    q: 'Qual enzima libera o ácido araquidônico dos fosfolipídios de membrana?',
    options: ['Ciclo-oxigenase', 'Fosfolipase A₂', 'Tromboxano sintase', 'Citocromo P450'],
    answer: 1,
    why: 'A fosfolipase A₂ hidrolisa fosfolipídios e libera o ácido araquidônico, substrato das ciclo-oxigenases.'
  },
  {
    q: 'Como o ibuprofeno inibe as ciclo-oxigenases?',
    options: [
      'Acetilação irreversível de um resíduo de serina',
      'Inibição reversível e não seletiva de COX-1 e COX-2',
      'Ligação ao receptor intracelular de glicocorticoides',
      'Bloqueio da fosfolipase A₂'
    ],
    answer: 1,
    why: 'O ibuprofeno compete com o ácido araquidônico no canal da enzima e se liga de forma reversível, inibindo COX-1 e COX-2.'
  },
  {
    q: 'A inibição da COX-1 ajuda a explicar qual efeito adverso dos AINEs?',
    options: [
      'Lesão e sangramento gastrointestinal',
      'Hipoglicemia',
      'Hipertricose',
      'Bradicardia por bloqueio beta'
    ],
    answer: 0,
    why: 'Prostaglandinas derivadas da COX-1 estimulam muco, bicarbonato e fluxo sanguíneo na mucosa gástrica. Ao reduzi-las, aumenta o risco de lesão.'
  },
  {
    q: 'Qual afirmação sobre os corticosteroides está correta?',
    options: [
      'São inibidores diretos e reversíveis da COX, como o ibuprofeno',
      'Ligam-se a um receptor intracelular e modulam a expressão gênica',
      'Atuam apenas no estômago',
      'Não têm efeito sobre citocinas'
    ],
    answer: 1,
    why: 'Os glicocorticoides atravessam a membrana, ligam-se ao receptor de glicocorticoides e alteram a transcrição de genes, reduzindo COX-2, citocinas e outros mediadores.'
  },
  {
    q: 'O ibuprofeno circula cerca de 99% ligado à albumina. O que isso implica?',
    options: [
      'Toda a dose circulante está ativa',
      'Apenas a fração livre difunde aos tecidos e é farmacologicamente ativa',
      'O fármaco não chega ao fígado',
      'O fármaco não é eliminado'
    ],
    answer: 1,
    why: 'A fração ligada funciona como reservatório; a fração livre atravessa o endotélio, alcança os alvos e também pode ser metabolizada e eliminada.'
  },
  {
    q: 'O que diferencia o ácido acetilsalicílico dos demais AINEs na ação sobre a COX?',
    options: [
      'É um corticosteroide',
      'Inibe a enzima de forma irreversível, por acetilação',
      'Não inibe a COX-1',
      'Atua apenas no sistema nervoso central'
    ],
    answer: 1,
    why: 'A acetilação é covalente. Como a plaqueta não sintetiza nova COX, o efeito antiplaquetário persiste por 7 a 10 dias.'
  },
  {
    q: 'Qual orientação é adequada ao administrar ibuprofeno por via oral?',
    options: [
      'Oferecer com água (e com alimento se houver desconforto gástrico) e relatar fezes escuras ou dor epigástrica',
      'Administrar com o paciente deitado, sem líquido',
      'Associar outro AINE para potencializar o efeito',
      'Não considerar a função renal'
    ],
    answer: 0,
    why: 'Água e posição adequada reduzem a irritação esofágica; sinais de sangramento digestivo devem ser comunicados e a função renal deve ser considerada.'
  }
]

const REFS = [
  'Rang & Dale. Farmacologia.',
  'Goodman & Gilman. As Bases Farmacológicas da Terapêutica.',
  'Katzung & Trevor. Farmacologia Básica e Clínica.',
  'Bula profissional e monografia oficial do ibuprofeno, conforme a autoridade regulatória aplicável.',
  'Artigos de revisão e diretrizes clínicas sobre anti-inflamatórios não esteroidais, ciclo-oxigenases e segurança gastrointestinal, cardiovascular e renal.'
]

/* ==========================================================================
   AUXILIARES
   ========================================================================== */

const hexToRgb = (h) => [1, 3, 5].map((i) => parseInt(h.slice(i, i + 2), 16))

const mix = (a, b, t) => {
  const A = hexToRgb(a)
  const B = hexToRgb(b)
  return `rgb(${A.map((v, i) => Math.round(v + (B[i] - v) * t)).join(',')})`
}

const glow = (active) => ({
  opacity: active ? 1 : 0.88,
  filter: active ? 'drop-shadow(0 0 9px #28c5e5)' : 'none',
  transition: 'opacity 0.3s, filter 0.3s'
})

function useReveal() {
  useEffect(() => {
    const els = document.querySelectorAll('.reveal')
    const io = new IntersectionObserver(
      (entries) =>
        entries.forEach((e) => {
          if (e.isIntersecting) {
            e.target.classList.add('in')
            io.unobserve(e.target)
          }
        }),
      { threshold: 0.1 }
    )
    els.forEach((el) => io.observe(el))
    return () => io.disconnect()
  }, [])
}

function Ibu() {
  return (
    <g>
      <polygon
        points="-6,-3.5 0,-7 6,-3.5 6,3.5 0,7 -6,3.5"
        fill="#28c5e5"
        stroke="#a1edfa"
        strokeWidth="1"
      />
      <path
        d="M6 0 L13 0 M13 0 L17 -4 M13 0 L17 4"
        stroke="#a1edfa"
        strokeWidth="1.5"
        fill="none"
        strokeLinecap="round"
      />
    </g>
  )
}

/* ==========================================================================
   NAVEGAÇÃO
   ========================================================================== */

function Navbar() {
  const [active, setActive] = useState('inicio')
  const [open, setOpen] = useState(false)
  const bar = useRef(null)

  useEffect(() => {
    const els = NAV.map(([id]) => document.getElementById(id)).filter(Boolean)
    const io = new IntersectionObserver(
      (entries) =>
        entries.forEach((e) => {
          if (e.isIntersecting) setActive(e.target.id)
        }),
      { rootMargin: '-40% 0px -55% 0px' }
    )
    els.forEach((el) => io.observe(el))
    return () => io.disconnect()
  }, [])

  useEffect(() => {
    const onScroll = () => {
      const h = document.documentElement
      const max = h.scrollHeight - h.clientHeight
      const p = max > 0 ? h.scrollTop / max : 0
      if (bar.current) bar.current.style.transform = `scaleX(${p})`
    }
    onScroll()
    window.addEventListener('scroll', onScroll, { passive: true })
    return () => window.removeEventListener('scroll', onScroll)
  }, [])

  return (
    <header className="navbar">
      <a className="logo" href="#inicio">
        INFLAMMA<span>.</span>
      </a>
      <button
        className="nav-toggle"
        aria-label="Abrir menu"
        aria-expanded={open}
        onClick={() => setOpen((o) => !o)}
      >
        <span></span>
        <span></span>
        <span></span>
      </button>
      <nav className={open ? 'nav-links open' : 'nav-links'}>
        {NAV.map(([id, label]) => (
          <a
            key={id}
            href={`#${id}`}
            className={active === id ? 'active' : ''}
            onClick={() => setOpen(false)}
          >
            {label}
          </a>
        ))}
      </nav>
      <span className="edition">PHARMACOLOGY / 001</span>
      <div className="scroll-progress" ref={bar}></div>
    </header>
  )
}

/* ==========================================================================
   FARMACOCINÉTICA
   ========================================================================== */

/* Árvore vascular: artérias (vermelho), veias (azul) e veia porta (violeta) */
const Vascular = memo(function Vascular({ hl }) {
  return (
    <g className={hl ? 'vessels hl' : 'vessels'}>
      <g className="v-casing">
        {V_TRUNKS.map((v, i) => (
          <path key={i} d={v.d} />
        ))}
      </g>
      <g className="v-thin">
        {V_THIN.map((v, i) =>
          v.t === 2 ? (
            <path key={i} d={v.d} className={`vf vf-${v.kind}`} style={{ opacity: v.a }} />
          ) : (
            <path
              key={i}
              d={v.d}
              className={`vs vs-${v.kind}`}
              strokeWidth={v.w}
              style={{ opacity: v.a }}
            />
          )
        )}
      </g>
      <g className="v-trunks">
        {V_TRUNKS.map((v, i) => (
          <path key={i} d={v.d} className={`vf vf-${v.kind}`} />
        ))}
      </g>
      <g className="v-hi">
        {V_TRUNKS.map((v, i) => (
          <path
            key={i}
            d={v.c}
            className={v.flow ? `vflow vflow-${v.kind}` : `vhi vhi-${v.kind}`}
            style={{ strokeWidth: Math.min(1.5, v.w * 0.36) }}
          />
        ))}
      </g>
    </g>
  )
})

function Pharmacokinetics({ jump }) {
  const N = STAGES.length
  const [tab, setTab] = useState('journey')
  const [selected, setSelected] = useState('stomach')
  const [stage, setStage] = useState(0)
  const [mode, setMode] = useState('idle')
  const [speed, setSpeed] = useState(1)

  const stageRef = useRef(0)
  const tRef = useRef(0)
  const speedRef = useRef(1)
  const trailRefs = useRef([])
  const lensRef = useRef([])
  const pillRef = useRef(null)
  const pillShapeRef = useRef(null)

  const apply = (s, t) => {
    const lens = lensRef.current
    if (!lens.length) return
    trailRefs.current.forEach((p, i) => {
      if (!p) return
      const L = lens[i]
      const shown = i < s ? L : i === s ? L * t : 0
      p.style.strokeDasharray = `${shown} ${L + 1}`
    })
    const seg = trailRefs.current[s]
    if (seg && pillRef.current) {
      const pt = seg.getPointAtLength(lens[s] * t)
      pillRef.current.setAttribute('transform', `translate(${pt.x} ${pt.y})`)
    }
    const shape = pillShapeRef.current
    if (shape) {
      const d = s < 2 ? 0 : s === 2 ? t : 1
      shape.setAttribute('rx', String(6 - 3.4 * d))
      shape.setAttribute('ry', String(3.6 - 1.0 * d))
      shape.setAttribute('fill', mix('#e9f0f7', '#28c5e5', d))
    }
  }

  useEffect(() => {
    lensRef.current = trailRefs.current.map((p) => (p ? p.getTotalLength() : 0))
    apply(0, 0)
  }, [])

  useEffect(() => {
    if (mode === 'idle') return undefined
    let raf
    let last = performance.now()
    const reduced =
      typeof window !== 'undefined' &&
      window.matchMedia &&
      window.matchMedia('(prefers-reduced-motion: reduce)').matches
    const tick = (now) => {
      const dt = Math.min(now - last, 64)
      last = now
      let s = stageRef.current
      let t = tRef.current + (reduced ? 1 : (dt * speedRef.current) / DUR[s])
      if (t >= 1) {
        if (mode === 'all' && s < N - 1) {
          s += 1
          t = 0
          stageRef.current = s
          setStage(s)
        } else {
          tRef.current = 1
          apply(s, 1)
          setMode('idle')
          return
        }
      }
      tRef.current = t
      apply(s, t)
      raf = requestAnimationFrame(tick)
    }
    raf = requestAnimationFrame(tick)
    return () => cancelAnimationFrame(raf)
  }, [mode])

  const goTo = (i, auto = 'step') => {
    stageRef.current = i
    tRef.current = 0
    setStage(i)
    apply(i, 0)
    setMode(auto)
  }

  useEffect(() => {
    if (jump && jump.stage != null) {
      setTab('journey')
      goTo(jump.stage)
    }
  }, [jump])

  const next = () => {
    if (tRef.current < 1) {
      setMode('step')
      return
    }
    if (stageRef.current < N - 1) goTo(stageRef.current + 1)
  }

  const prev = () => {
    if (stageRef.current > 0) goTo(stageRef.current - 1)
  }

  const playAll = () => {
    if (mode !== 'idle') {
      setMode('idle')
      return
    }
    let s = stageRef.current
    if (tRef.current >= 1) {
      s = s >= N - 1 ? 0 : s + 1
      stageRef.current = s
      tRef.current = 0
      setStage(s)
      apply(s, 0)
    }
    setMode('all')
  }

  const toggleSpeed = () => {
    const v = speed === 1 ? 0.5 : 1
    speedRef.current = v
    setSpeed(v)
  }

  const current = STAGES[stage]
  const hk =
    tab === 'journey'
      ? current.key === 'portal'
        ? 'blood'
        : current.key
      : selected

  return (
    <section className="section" id="farmacocinetica">
      <div className="section-heading reveal">
        <span className="section-number">02 — FARMACOCINÉTICA</span>
        <h2>O percurso do medicamento.</h2>
        <p>
          Acompanhe o ibuprofeno da boca ao tecido inflamado e, depois, até a
          eliminação. Reproduza tudo, pause ou avance etapa por etapa.
        </p>
      </div>

      <div className="pk-layout reveal">
        <div className="body-illustration">
          <svg
            viewBox="0 0 420 620"
            role="img"
            aria-label="Ilustração esquemática do corpo humano, do sistema digestório e do percurso do comprimido"
          >
            <defs>
              <radialGradient id="bodyGlow">
                <stop offset="0%" stopColor="#1c5366" stopOpacity=".35" />
                <stop offset="100%" stopColor="#0a131d" stopOpacity="0" />
              </radialGradient>
              <linearGradient id="bodyFill" x1="0" y1="0" x2="1" y2="1">
                <stop offset="0%" stopColor="#254455" />
                <stop offset="100%" stopColor="#122735" />
              </linearGradient>
              <linearGradient id="organFill" x1="0" y1="0" x2="1" y2="1">
                <stop offset="0%" stopColor="#e78b88" />
                <stop offset="100%" stopColor="#a94755" />
              </linearGradient>
              <linearGradient id="heartFill" x1="0" y1="0" x2="1" y2="1">
                <stop offset="0%" stopColor="#d9475a" />
                <stop offset="100%" stopColor="#8a2c3d" />
              </linearGradient>
            </defs>

            <ellipse cx="210" cy="310" rx="195" ry="290" fill="url(#bodyGlow)" />

            {/* Pernas */}
            <path
              d="M134 450 L124 578 L120 596 Q118 606 134 606 L178 606 L182 580 L206 490 L210 474 Z
                 M286 450 L296 578 L300 596 Q302 606 286 606 L242 606 L238 580 L214 490 L210 474 Z"
              fill="#18303e"
              stroke="#52798d"
              strokeWidth="2"
              strokeLinejoin="round"
            />

            {/* Braços */}
            <path
              d="M122 158 C104 166 98 196 92 232 L72 322 Q68 340 84 346 L104 352 L128 262 Z
                 M298 158 C316 166 322 196 328 232 L348 322 Q352 340 336 346 L316 352 L292 262 Z"
              fill="url(#bodyFill)"
              stroke="#52798d"
              strokeWidth="2"
              strokeLinejoin="round"
            />

            {/* Tronco */}
            <path
              d="M190 128 C160 130 130 138 120 160 L116 330 Q116 440 134 474 L286 474 Q304 440 304 330 L300 160 C290 138 260 130 230 128 Z"
              fill="url(#bodyFill)"
              stroke="#52798d"
              strokeWidth="2"
              strokeLinejoin="round"
            />

            {/* Pescoço e cabeça */}
            <path
              d="M192 108 L192 132 L228 132 L228 108"
              fill="url(#bodyFill)"
              stroke="#52798d"
              strokeWidth="2"
            />
            <path
              d="M210 24
                 C179 24 166 47 168 77
                 C169 105 185 119 210 119
                 C235 119 251 105 252 77
                 C254 47 241 24 210 24Z"
              fill="url(#bodyFill)"
              stroke="#648ba0"
              strokeWidth="2"
            />

            {/* Clavículas */}
            <path
              d="M210 142 Q177 139 148 159 M210 142 Q243 139 272 159"
              stroke="#62899b"
              strokeWidth="2"
              fill="none"
              opacity=".6"
            />

            {/* Pulmões */}
            <path
              d="M200 160 C178 148 146 162 141 198 C137 228 144 250 172 253 C192 255 200 244 200 232 Z"
              fill="#2d5f76"
              stroke="#6fa0b6"
              strokeWidth="1.5"
              opacity=".85"
            />
            <path
              d="M220 160 C242 148 274 162 279 198 C283 228 276 250 248 253 C228 255 220 244 220 232 Z"
              fill="#2d5f76"
              stroke="#6fa0b6"
              strokeWidth="1.5"
              opacity=".85"
            />

            {/* Coração */}
            <path
              d="M209 200 C206 212 208 224 216 232 C222 238 229 241 235 242 C243 236 248 226 246 212 C245 200 240 192 232 192 C227 192 224 195 222 198 C218 193 212 194 209 200Z"
              fill="url(#heartFill)"
              stroke="#e07a86"
              strokeWidth="1.5"
              strokeLinejoin="round"
            />
            <path
              d="M226 203 C236 201 242 209 241 221 M214 211 C216 221 222 229 231 234"
              stroke="#ffb3b9"
              strokeWidth="1.3"
              strokeLinecap="round"
              fill="none"
              opacity=".4"
            />

            {/* Traqueia e brônquios */}
            <path
              d="M205 120 L205 176 M205 176 L192 194 M205 176 L218 192"
              stroke="#8eb1c1"
              strokeWidth="5"
              strokeLinecap="round"
              strokeLinejoin="round"
              fill="none"
              opacity=".85"
            />

            {/* Esôfago */}
            <g style={glow(hk === 'esophagus')}>
              <path
                d="M210 100 L210 112 L212 124 L214 134 L214 252 Q216 266 232 269"
                stroke="#d98d86"
                strokeWidth="7"
                strokeLinecap="round"
                strokeLinejoin="round"
                fill="none"
              />
              <path
                d="M210 100 L210 112 L212 124 L214 134 L214 252 Q216 266 232 269"
                stroke="#f0b8ae"
                strokeWidth="1.5"
                strokeLinecap="round"
                strokeLinejoin="round"
                fill="none"
                opacity=".7"
              />
            </g>

            {/* Diafragma */}
            <path
              d="M130 256 C150 246 190 244 210 252 C232 244 270 246 290 258"
              stroke="#7fb0c4"
              strokeWidth="1.5"
              strokeDasharray="4 4"
              fill="none"
              opacity=".6"
            />

            {/* Rins */}
            <g style={glow(hk === 'kidney')}>
              <ellipse
                cx="152"
                cy="338"
                rx="10"
                ry="20"
                transform="rotate(10 152 338)"
                fill="#8b4552"
                stroke="#c9707c"
                strokeWidth="1.5"
              />
              <ellipse
                cx="270"
                cy="326"
                rx="10"
                ry="20"
                transform="rotate(-10 270 326)"
                fill="#8b4552"
                stroke="#c9707c"
                strokeWidth="1.5"
              />
            </g>

            {/* Pâncreas */}
            <path
              d="M184 334 Q214 322 252 336 Q258 344 246 349 Q214 356 184 346Z"
              fill="#d8a46d"
              stroke="#efc18a"
              strokeWidth="1.5"
            />

            {/* Intestino grosso */}
            <g style={glow(false)}>
              <path
                d="M158 436 C148 436 146 424 146 412 L146 376 C146 356 162 354 180 360 C202 368 220 368 242 360 C260 354 276 356 276 376 L276 420 C276 440 262 446 244 444 C228 442 222 450 214 456"
                fill="none"
                stroke="#e9a08b"
                strokeWidth="15"
                strokeLinecap="round"
                strokeLinejoin="round"
              />
              <path
                d="M214 456 L211 470"
                stroke="#e9a08b"
                strokeWidth="10"
                strokeLinecap="round"
              />
            </g>

            {/* Duodeno e intestino delgado */}
            <g style={glow(hk === 'intestine')}>
              <path
                d="M214 314 C196 318 188 334 192 350 C196 366 206 374 214 380"
                fill="none"
                stroke="#d98273"
                strokeWidth="8"
                strokeLinecap="round"
                strokeLinejoin="round"
              />
              <path
                d={SMALL_INTESTINE}
                fill="none"
                stroke="#d98273"
                strokeWidth="8"
                strokeLinecap="round"
                strokeLinejoin="round"
              />
            </g>

            {/* Fígado */}
            <g style={glow(hk === 'liver')}>
              <path
                d="M138 262 C150 252 176 252 198 258 Q208 262 206 276 Q200 296 178 300 C156 304 136 296 130 282 Q128 270 138 262Z"
                fill="url(#organFill)"
                stroke={hk === 'liver' ? '#f6d4ca' : '#f3a09a'}
                strokeWidth="2"
              />
              <path
                d="M170 259 Q172 276 160 294"
                stroke="#f6b2a8"
                strokeWidth="2"
                fill="none"
                opacity=".6"
              />
            </g>

            {/* Estômago em J */}
            <g style={glow(hk === 'stomach')}>
              <path
                d="M232 262 C248 252 272 258 278 278 C284 302 274 326 252 328 C236 330 222 324 214 316 L214 304 C228 306 240 302 244 290 C246 280 238 276 230 276 Z"
                fill="url(#organFill)"
                stroke={hk === 'stomach' ? '#f6d4ca' : '#f3a09a'}
                strokeWidth="2"
                strokeLinejoin="round"
              />
              <path
                d="M252 272 Q268 290 262 312"
                stroke="#f6b2a8"
                strokeWidth="2"
                fill="none"
                opacity=".55"
              />
            </g>

            {/* Árvore vascular */}
            <Vascular hl={hk === 'blood' || hk === 'tissue' || hk === 'kidney'} />

            {/* Tecido inflamado (joelho) */}
            <g style={glow(hk === 'tissue')}>
              <circle
                className="inflamed-pulse"
                cx="258"
                cy="520"
                r="22"
                fill="#e5484d"
                opacity=".2"
              />
              <circle cx="258" cy="520" r="11" fill="#e5484d" opacity=".4" />
              <path
                d="M246 512 Q258 504 270 512 M245 528 Q258 538 271 528"
                stroke="#ff8f8f"
                strokeWidth="1.5"
                fill="none"
                opacity=".8"
              />
              <text x="304" y="516" className="svg-label tissue-label">
                TECIDO
              </text>
              <text x="304" y="528" className="svg-label tissue-label">
                INFLAMADO
              </text>
            </g>

            {/* Boca */}
            <ellipse
              cx="210"
              cy="92"
              rx="11"
              ry="4"
              fill="#7a2f3c"
              stroke="#e7a49a"
              strokeWidth="1.5"
              style={glow(hk === 'mouth')}
            />

            {/* Percurso */}
            <g
              className="route"
              fill="none"
              style={{ opacity: tab === 'journey' ? 1 : 0 }}
            >
              {ROUTE.map((d, i) => (
                <path key={`b${i}`} d={d} className="route-base" />
              ))}
              {ROUTE.map((d, i) => (
                <path
                  key={`t${i}`}
                  d={d}
                  className="route-trail"
                  ref={(el) => {
                    trailRefs.current[i] = el
                  }}
                />
              ))}
            </g>

            {/* Comprimido / fármaco */}
            <g
              ref={pillRef}
              transform="translate(210 92)"
              style={{ opacity: tab === 'journey' ? 1 : 0, transition: 'opacity .3s' }}
            >
              <ellipse
                ref={pillShapeRef}
                className="pill-shape"
                rx="6"
                ry="3.6"
                fill="#e9f0f7"
                stroke="#28c5e5"
                strokeWidth="1.2"
              />
            </g>
          </svg>
          <div className="vessel-legend" aria-hidden="true">
            <span>
              <i className="lg lg-a"></i>Artérias
            </span>
            <span>
              <i className="lg lg-v"></i>Veias
            </span>
            <span>
              <i className="lg lg-p"></i>Veia porta
            </span>
          </div>
          <div className="anatomy-caption">
            REPRESENTAÇÃO ANATÔMICA ESQUEMÁTICA · NÃO SUBSTITUI UM ATLAS
          </div>
        </div>

        <div className="pk-panel">
          <div className="tabs" role="tablist">
            <button
              role="tab"
              aria-selected={tab === 'journey'}
              className={tab === 'journey' ? 'tab-btn active' : 'tab-btn'}
              onClick={() => setTab('journey')}
            >
              Percurso do comprimido
            </button>
            <button
              role="tab"
              aria-selected={tab === 'explore'}
              className={tab === 'explore' ? 'tab-btn active' : 'tab-btn'}
              onClick={() => setTab('explore')}
            >
              Explorar estruturas
            </button>
          </div>

          {tab === 'journey' ? (
            <>
              <div className="adme-chips" aria-label="Fase farmacocinética atual">
                {ADME.map((a) => (
                  <span
                    key={a.k}
                    className={a.on.includes(stage) ? 'chip on' : 'chip'}
                    title={a.name}
                  >
                    <b>{a.k}</b> {a.name}
                  </span>
                ))}
              </div>

              <div className="stage-dots">
                {STAGES.map((s, i) => (
                  <button
                    key={s.key}
                    className={`dot ${i === stage ? 'active' : ''} ${i < stage ? 'past' : ''}`}
                    onClick={() => goTo(i)}
                    aria-label={`Etapa ${i + 1}: ${s.title}`}
                    title={s.title}
                  >
                    {i + 1}
                  </button>
                ))}
              </div>

              <div className="organ-description" aria-live="polite">
                <span>
                  ETAPA {stage + 1} DE {N} · {current.phase.toUpperCase()}
                </span>
                <h3>{current.title}</h3>
                <p>{current.text}</p>
                {current.nursing && (
                  <p className="nursing">
                    <strong>Enfermagem:</strong> {current.nursing}
                  </p>
                )}
              </div>

              <div className="controls">
                <button className="ctrl-btn" onClick={prev} disabled={stage === 0}>
                  ◀ Anterior
                </button>
                <button className="ctrl-btn primary" onClick={playAll}>
                  {mode !== 'idle' ? '❚❚ Pausar' : '▶ Reproduzir tudo'}
                </button>
                <button className="ctrl-btn" onClick={next}>
                  Próxima ▶
                </button>
                <button className="ctrl-btn" onClick={() => goTo(0, 'idle')}>
                  ↺ Reiniciar
                </button>
                <button className="ctrl-btn" onClick={toggleSpeed}>
                  Velocidade {speed === 1 ? '1×' : '0,5×'}
                </button>
              </div>
            </>
          ) : (
            <>
              <span className="section-number">EXPLORAÇÃO INTERATIVA</span>
              {Object.entries(ORGANS).map(([key, organ], i) => (
                <button
                  key={key}
                  className={`organ-button ${selected === key ? 'active' : ''}`}
                  onClick={() => setSelected(key)}
                >
                  <span className="organ-number">{String(i + 1).padStart(2, '0')}</span>
                  <span>{organ.name}</span>
                  <span className="organ-arrow">↗</span>
                </button>
              ))}
              <div className="organ-description">
                <span>ESTRUTURA SELECIONADA</span>
                <h3>{ORGANS[selected].name}</h3>
                <p>{ORGANS[selected].description}</p>
              </div>
            </>
          )}
        </div>
      </div>

      <div className="adme-grid reveal">
        {ADME_CARDS.map((c) => (
          <div className="adme-card" key={c.letter}>
            <span className="adme-letter">{c.letter}</span>
            <h3>{c.title}</h3>
            <p>{c.text}</p>
          </div>
        ))}
      </div>
      <p className="fineprint reveal">
        Valores aproximados, para fins educacionais. Confira sempre a bula e a
        bibliografia de referência antes de usar o conteúdo como material didático
        definitivo.
      </p>
    </section>
  )
}

/* ==========================================================================
   DO TECIDO À CÉLULA (zoom)
   ========================================================================== */

function ScaleZoom() {
  const [level, setLevel] = useState(0)
  const [auto, setAuto] = useState(false)

  useEffect(() => {
    if (!auto) return undefined
    const id = setInterval(() => setLevel((l) => Math.min(l + 1, 2)), 3200)
    return () => clearInterval(id)
  }, [auto])

  useEffect(() => {
    if (auto && level >= 2) setAuto(false)
  }, [auto, level])

  const startAuto = () => {
    setLevel(0)
    setAuto(true)
  }

  const lipids = Array.from({ length: 19 }, (_, i) => 30 + i * 30)

  return (
    <section className="section" id="zoom">
      <div className="section-heading reveal">
        <span className="section-number">03 — MUDANÇA DE ESCALA</span>
        <h2>Do tecido à célula.</h2>
        <p>
          Depois de se distribuir, o fármaco chega ao local da inflamação. Aproxime
          a câmera: tecido, célula e, por fim, a membrana e suas enzimas.
        </p>
      </div>

      <div className="zoom-layout reveal">
        <div className="zoom-stage">
          <svg viewBox="0 0 600 380" role="img" aria-label="Aproximação do tecido inflamado até a membrana da célula">
            {/* Nível 1: tecido */}
            <g className={level === 0 ? 'zoom-level on' : 'zoom-level'}>
              <rect
                x="20"
                y="46"
                width="560"
                height="66"
                rx="33"
                fill="#3a1420"
                stroke="#c9707c"
                strokeWidth="3"
              />
              {[70, 160, 250, 340, 430, 520].map((x) => (
                <ellipse key={x} cx={x} cy="76" rx="15" ry="8" fill="#c0394b" />
              ))}
              {[110, 205, 295, 385, 475].map((x) => (
                <circle key={x} cx={x} cy="96" r="4" fill="#28c5e5" />
              ))}
              <path
                d="M300 114 L300 148 M330 114 L332 150"
                stroke="#e5484d"
                strokeWidth="2"
                strokeDasharray="3 4"
                fill="none"
              />
              {[
                [304, 130],
                [334, 134],
                [296, 152],
                [338, 158]
              ].map(([x, y]) => (
                <circle key={`${x}-${y}`} cx={x} cy={y} r="3.5" fill="#28c5e5" />
              ))}
              {[
                [80, 230, 30],
                [170, 300, 24],
                [470, 240, 34],
                [530, 320, 24],
                [400, 320, 22]
              ].map(([x, y, r]) => (
                <circle
                  key={`${x}-${y}`}
                  cx={x}
                  cy={y}
                  r={r}
                  fill="#14313f"
                  stroke="#2b5a70"
                  strokeWidth="2"
                />
              ))}
              <circle
                cx="300"
                cy="256"
                r="46"
                fill="#16405a"
                stroke="#28c5e5"
                strokeWidth="3"
                className="inflamed-pulse"
              />
              <circle cx="300" cy="256" r="16" fill="#0b2838" stroke="#35d8f3" />
              {[
                [240, 190],
                [370, 196],
                [250, 330],
                [360, 330],
                [200, 200]
              ].map(([x, y]) => (
                <circle key={`${x}-${y}`} cx={x} cy={y} r="4" fill="#5b8aa5" opacity=".6" />
              ))}
              <text x="30" y="36" className="svg-label">
                CAPILAR DILATADO · FRAÇÃO LIVRE DO FÁRMACO
              </text>
              <text x="342" y="146" className="svg-label tissue-label">
                ↑ PERMEABILIDADE
              </text>
              <text x="360" y="262" className="svg-label">
                CÉLULA-ALVO
              </text>
              <text x="30" y="360" className="svg-label tissue-label">
                EDEMA NO INTERSTÍCIO
              </text>
            </g>

            {/* Nível 2: célula */}
            <g className={level === 1 ? 'zoom-level on' : 'zoom-level'}>
              <circle cx="320" cy="190" r="150" fill="#12303f" stroke="#35d8f3" strokeWidth="6" />
              <circle
                cx="320"
                cy="190"
                r="143"
                fill="none"
                stroke="#1c7890"
                strokeWidth="1"
                strokeDasharray="3 5"
              />
              <circle cx="355" cy="165" r="48" fill="#0b2838" stroke="#4f7cff" strokeWidth="3" />
              <circle cx="355" cy="165" r="14" fill="#1b3c66" />
              <path
                d="M250 235 C270 212 290 252 310 232 S350 252 372 240 C392 230 404 250 416 258"
                fill="none"
                stroke="#4f7cff"
                strokeWidth="5"
                opacity=".85"
              />
              <path
                d="M262 262 C282 244 300 276 320 262 S360 276 384 266"
                fill="none"
                stroke="#4f7cff"
                strokeWidth="4"
                opacity=".6"
              />
              <ellipse cx="260" cy="150" rx="22" ry="11" fill="#3b6b5e" stroke="#6fb59f" />
              <ellipse cx="420" cy="215" rx="20" ry="10" fill="#3b6b5e" stroke="#6fb59f" />
              {[
                [60, 150],
                [92, 200],
                [64, 240]
              ].map(([x, y]) => (
                <circle key={`${x}-${y}`} cx={x} cy={y} r="5" fill="#28c5e5" />
              ))}
              <path
                d="M110 190 L160 190"
                stroke="#28c5e5"
                strokeWidth="2"
                strokeDasharray="4 4"
                markerEnd="none"
                className="flow-line"
              />
              {[
                [214, 170],
                [226, 214]
              ].map(([x, y]) => (
                <circle key={`${x}-${y}`} cx={x} cy={y} r="5" fill="#28c5e5" />
              ))}
              <text x="30" y="30" className="svg-label">
                FÁRMACO NO INTERSTÍCIO
              </text>
              <text x="330" y="34" className="svg-label">
                MEMBRANA PLASMÁTICA
              </text>
              <text x="332" y="170" className="svg-label">
                NÚCLEO
              </text>
              <text x="260" y="290" className="svg-label">
                RETÍCULO ENDOPLASMÁTICO · COX
              </text>
              <text x="236" y="150" className="svg-label">
                CITOPLASMA
              </text>
            </g>

            {/* Nível 3: membrana e enzimas */}
            <g className={level === 2 ? 'zoom-level on' : 'zoom-level'}>
              {lipids.map((x) => (
                <g key={x}>
                  <circle cx={x} cy="110" r="10" fill="#3aa0b9" />
                  <path
                    d={`M${x - 4} 120 L${x - 4} 160 M${x + 4} 120 L${x + 4} 160`}
                    stroke="#6fc3d6"
                    strokeWidth="2"
                  />
                  <circle cx={x} cy="210" r="10" fill="#3aa0b9" />
                  <path
                    d={`M${x - 4} 200 L${x - 4} 160 M${x + 4} 200 L${x + 4} 160`}
                    stroke="#6fc3d6"
                    strokeWidth="2"
                  />
                </g>
              ))}
              <ellipse cx="300" cy="244" rx="44" ry="20" fill="#2a4f9a" stroke="#4f7cff" strokeWidth="2" />
              <text x="300" y="249" textAnchor="middle" className="svg-label strong">
                PLA₂
              </text>
              <rect x="190" y="290" width="80" height="60" rx="16" fill="#1b3c66" stroke="#4f7cff" strokeWidth="2" />
              <rect x="280" y="290" width="80" height="60" rx="16" fill="#1b3c66" stroke="#4f7cff" strokeWidth="2" />
              <rect x="224" y="296" width="14" height="40" rx="7" fill="#07111a" stroke="#7fb0c4" />
              <rect x="314" y="296" width="14" height="40" rx="7" fill="#07111a" stroke="#7fb0c4" />
              <text x="30" y="80" className="svg-label">
                MEIO EXTRACELULAR
              </text>
              <text x="30" y="260" className="svg-label">
                CITOPLASMA
              </text>
              <text x="372" y="324" className="svg-label">
                COX-1 / COX-2
              </text>
              <text x="30" y="150" className="svg-label strong">
                BICAMADA DE FOSFOLIPÍDIOS
              </text>
            </g>
          </svg>
        </div>

        <div className="zoom-info">
          <div className="tabs" role="tablist">
            {ZOOM.map((z, i) => (
              <button
                key={z.tab}
                role="tab"
                aria-selected={level === i}
                className={level === i ? 'tab-btn active' : 'tab-btn'}
                onClick={() => {
                  setAuto(false)
                  setLevel(i)
                }}
              >
                {z.tab}
              </button>
            ))}
          </div>
          <div className="organ-description" aria-live="polite">
            <span>NÍVEL {level + 1} DE 3</span>
            <h3>{ZOOM[level].title}</h3>
            <p>{ZOOM[level].text}</p>
          </div>
          <div className="controls">
            <button className="ctrl-btn primary" onClick={startAuto}>
              ▶ Aproximar automaticamente
            </button>
            {level === 2 && (
              <a className="ctrl-btn" href="#farmacodinamica">
                Ir ao mecanismo molecular ↓
              </a>
            )}
          </div>
        </div>
      </div>
    </section>
  )
}

/* ==========================================================================
   FARMACODINÂMICA
   ========================================================================== */

function Pharmacodynamics() {
  const [step, setStep] = useState(0)
  const [drug, setDrug] = useState(false)
  const [auto, setAuto] = useState(false)

  useEffect(() => {
    if (!auto) return undefined
    const id = setInterval(() => setStep((s) => Math.min(s + 1, PD_STEPS.length - 1)), 3600)
    return () => clearInterval(id)
  }, [auto])

  useEffect(() => {
    if (auto && step >= PD_STEPS.length - 1) setAuto(false)
  }, [auto, step])

  const startAuto = () => {
    setStep(0)
    setAuto(true)
  }

  const lipids = Array.from({ length: 24 }, (_, i) => 22 + i * 22)

  /* AA: posição e rotação conforme a etapa e a presença do fármaco */
  let aaStyle
  if (step < 2) {
    aaStyle = { transform: 'translate(290px,180px) rotate(0deg)', opacity: 0 }
  } else if (step === 2) {
    aaStyle = { transform: 'translate(290px,235px) rotate(0deg)', opacity: 1 }
  } else if (drug) {
    aaStyle = { transform: 'translate(245px,250px) rotate(0deg)', opacity: step === 3 ? 1 : 0.5 }
  } else if (step === 3) {
    aaStyle = { transform: 'translate(245px,306px) rotate(90deg)', opacity: 1 }
  } else {
    aaStyle = { transform: 'translate(245px,306px) rotate(90deg)', opacity: 0 }
  }
  const aaTransition = { transition: 'transform 1.2s ease, opacity .8s' }

  const ibuPos = drug
    ? step >= 3
      ? [
          [245, 288],
          [335, 288],
          [480, 340]
        ]
      : [
          [470, 230],
          [500, 290],
          [430, 200]
        ]
    : [
        [640, 230],
        [640, 290],
        [640, 200]
      ]

  const chips = [
    { x: 20, name: 'PGE₂', fx: 'dor · febre · edema', fx2: 'mucosa · rim' },
    { x: 150, name: 'PGI₂', fx: 'vasodilatação', fx2: 'anti-agregante' },
    { x: 280, name: 'TXA₂', fx: 'agregação', fx2: 'plaquetária' },
    { x: 410, name: 'PGD₂ · PGF₂α', fx: 'mastócitos', fx2: 'músculo liso' }
  ]

  const chipsOn = step >= 4
  const chipsOpacity = chipsOn ? (drug ? 0.28 : 1) : 0

  return (
    <section className="section" id="farmacodinamica">
      <div className="section-heading reveal">
        <span className="section-number">04 — FARMACODINÂMICA</span>
        <h2>O que o fármaco faz na célula.</h2>
        <p>
          Siga a cascata do ácido araquidônico e veja, ao ativar o ibuprofeno,
          onde ele interfere e por que isso explica efeitos terapêuticos e
          adversos.
        </p>
      </div>

      <div className="pd-layout reveal">
        <div className="pd-visual">
          <svg viewBox="0 0 560 600" role="img" aria-label="Cascata do ácido araquidônico e inibição da COX pelo ibuprofeno">
            <defs>
              <linearGradient id="cytoGrad" x1="0" y1="0" x2="0" y2="1">
                <stop offset="0%" stopColor="#0f2a39" />
                <stop offset="100%" stopColor="#09141d" />
              </linearGradient>
            </defs>
            <rect x="0" y="167" width="560" height="433" fill="url(#cytoGrad)" />

            <text x="14" y="20" className="svg-label">MEIO EXTRACELULAR</text>
            <text x="14" y="196" className="svg-label">CITOPLASMA</text>
            <text x="546" y="86" textAnchor="end" className="svg-label">MEMBRANA PLASMÁTICA</text>
            <text x="14" y="296" className="svg-label">MEMBRANA DO RE / ENVOLTÓRIO NUCLEAR</text>

            {/* Estímulo */}
            <g className={step === 0 ? 'stimulus pulse-soft' : 'stimulus'}>
              <path
                d="M290 22 L278 48 L288 48 L282 72 L304 42 L292 42 L298 22Z"
                fill="#ffb86b"
              />
              <text x="318" y="46" className="svg-label">ESTÍMULO INFLAMATÓRIO / LESÃO</text>
            </g>

            {/* Membrana plasmática */}
            {lipids.map((x) => (
              <g key={x}>
                <circle cx={x} cy="100" r="7" fill="#3aa0b9" />
                <path d={`M${x - 3} 107 L${x - 3} 130 M${x + 3} 107 L${x + 3} 130`} stroke="#6fc3d6" strokeWidth="1.5" />
                <circle cx={x} cy="160" r="7" fill="#3aa0b9" />
                <path d={`M${x - 3} 153 L${x - 3} 130 M${x + 3} 153 L${x + 3} 130`} stroke="#6fc3d6" strokeWidth="1.5" />
              </g>
            ))}

            {/* Ca2+ */}
            <g style={{ opacity: step <= 1 ? 1 : 0, transition: 'opacity .6s' }}>
              {[
                [230, 215],
                [260, 228],
                [330, 222],
                [352, 206]
              ].map(([x, y]) => (
                <g key={`${x}-${y}`}>
                  <circle cx={x} cy={y} r="8" fill="#2a4f9a" stroke="#7fa2ff" />
                  <text x={x} y={y + 3} textAnchor="middle" className="svg-label strong tiny">Ca²⁺</text>
                </g>
              ))}
            </g>

            {/* PLA2 */}
            <g style={glow(step === 1)}>
              <ellipse cx="290" cy="187" rx="40" ry="20" fill="#2a4f9a" stroke="#4f7cff" strokeWidth="2" />
              <text x="290" y="192" textAnchor="middle" className="svg-label strong">PLA₂</text>
            </g>
            <path
              d="M278 144 L302 156"
              stroke="#e5484d"
              strokeWidth="3"
              strokeLinecap="round"
              style={{ opacity: step >= 1 ? 1 : 0, transition: 'opacity .5s' }}
            />

            {/* Membrana do RE e COX */}
            <rect x="110" y="304" width="340" height="30" rx="15" fill="#10293a" stroke="#1c7890" />
            <g style={glow(step === 3)}>
              <rect x="200" y="282" width="80" height="70" rx="18" fill={step === 3 ? '#24508a' : '#1b3c66'} stroke="#4f7cff" strokeWidth="2" />
              <rect x="290" y="282" width="80" height="70" rx="18" fill={step === 3 ? '#24508a' : '#1b3c66'} stroke="#4f7cff" strokeWidth="2" />
              <rect x="238" y="288" width="14" height="46" rx="7" fill="#07111a" stroke="#7fb0c4" />
              <rect x="328" y="288" width="14" height="46" rx="7" fill="#07111a" stroke="#7fb0c4" />
            </g>
            <text x="290" y="374" textAnchor="middle" className="svg-label strong">COX-1 / COX-2</text>
            <text x="290" y="388" textAnchor="middle" className="svg-label tiny">dímero na membrana do RE · COX-1 constitutiva · COX-2 induzível</text>

            {/* Ácido araquidônico */}
            <g style={{ ...aaStyle, ...aaTransition }}>
              <path
                d="M-24 4 l8 -8 l8 8 l8 -8 l8 8 l8 -8 l8 8"
                fill="none"
                stroke="#ffb86b"
                strokeWidth="3"
                strokeLinecap="round"
                strokeLinejoin="round"
              />
              <text x="0" y="-12" textAnchor="middle" className="svg-label tiny aa-label">ácido araquidônico</text>
            </g>

            {/* PGH2 */}
            <g style={{ opacity: step >= 3 && !drug ? 1 : 0, transition: 'opacity .8s' }}>
              <path d="M290 354 L290 400" className="flow-line" stroke="#28c5e5" strokeWidth="2" fill="none" />
              <rect x="226" y="400" width="128" height="30" rx="15" fill="#0b2838" stroke="#28c5e5" />
              <text x="290" y="419" textAnchor="middle" className="svg-label strong">PGG₂ → PGH₂</text>
            </g>

            {/* Setas e prostanoides */}
            <g style={{ opacity: chipsOpacity, transition: 'opacity .8s' }}>
              {chips.map((c) => (
                <path
                  key={c.name}
                  d={`M290 430 C290 450 ${c.x + 60} 446 ${c.x + 60} 466`}
                  className="flow-line"
                  stroke="#28c5e5"
                  strokeWidth="2"
                  fill="none"
                />
              ))}
              {chips.map((c) => (
                <g key={c.name}>
                  <rect x={c.x} y="466" width="120" height="64" rx="8" fill="#3a1420" stroke="#e5484d" strokeWidth="1.5" />
                  <text x={c.x + 60} y="490" textAnchor="middle" className="chip-title">{c.name}</text>
                  <text x={c.x + 60} y="508" textAnchor="middle" className="svg-label tiny">{c.fx}</text>
                  <text x={c.x + 60} y="520" textAnchor="middle" className="svg-label tiny">{c.fx2}</text>
                </g>
              ))}
            </g>
            <text
              x="290"
              y="560"
              textAnchor="middle"
              className="svg-label strong drug-result"
              style={{ opacity: drug && step >= 4 ? 1 : 0, transition: 'opacity .8s' }}
            >
              ↓ SÍNTESE DE PROSTANOIDES COM IBUPROFENO
            </text>

            {/* Ibuprofeno */}
            {ibuPos.map(([x, y], i) => (
              <g
                key={i}
                style={{
                  transform: `translate(${x}px,${y}px)`,
                  transition: 'transform 1.2s ease, opacity .6s',
                  opacity: drug ? 1 : 0
                }}
              >
                <Ibu />
              </g>
            ))}
            <text
              x="480"
              y="368"
              textAnchor="middle"
              className="svg-label strong"
              style={{ opacity: drug ? 1 : 0, transition: 'opacity .6s' }}
            >
              IBUPROFENO
            </text>

            {/* Bloqueio */}
            <g style={{ opacity: drug && step >= 3 ? 1 : 0, transition: 'opacity .6s' }}>
              <path d="M238 304 L252 318 M252 304 L238 318" stroke="#e5484d" strokeWidth="3.5" strokeLinecap="round" />
              <path d="M328 304 L342 318 M342 304 L328 318" stroke="#e5484d" strokeWidth="3.5" strokeLinecap="round" />
            </g>
          </svg>
        </div>

        <div className="pd-panel">
          <div className="switch-row">
            <button
              className={drug ? 'switch on' : 'switch'}
              role="switch"
              aria-checked={drug}
              onClick={() => setDrug((d) => !d)}
            >
              <span className="knob"></span>
            </button>
            <span>{drug ? 'Com ibuprofeno' : 'Sem fármaco: adicionar ibuprofeno'}</span>
          </div>

          <div className="stage-dots">
            {PD_STEPS.map((s, i) => (
              <button
                key={s.title}
                className={`dot ${i === step ? 'active' : ''} ${i < step ? 'past' : ''}`}
                onClick={() => {
                  setAuto(false)
                  setStep(i)
                }}
                aria-label={`Passo ${i + 1}: ${s.title}`}
                title={s.title}
              >
                {i + 1}
              </button>
            ))}
          </div>

          <div className="organ-description" aria-live="polite">
            <span>
              PASSO {step + 1} DE {PD_STEPS.length}
            </span>
            <h3>{PD_STEPS[step].title}</h3>
            <p>{PD_STEPS[step].text}</p>
            {drug && (
              <p className="drug-note">
                <strong>Com ibuprofeno:</strong> {PD_STEPS[step].drug}
              </p>
            )}
          </div>

          <div className="controls">
            <button className="ctrl-btn" onClick={() => { setAuto(false); setStep((s) => Math.max(0, s - 1)) }} disabled={step === 0}>
              ◀ Anterior
            </button>
            <button className="ctrl-btn primary" onClick={startAuto}>
              ▶ Reproduzir cascata
            </button>
            <button
              className="ctrl-btn"
              onClick={() => { setAuto(false); setStep((s) => Math.min(PD_STEPS.length - 1, s + 1)) }}
              disabled={step === PD_STEPS.length - 1}
            >
              Próximo ▶
            </button>
          </div>
        </div>
      </div>

      <div className="cox-cards reveal">
        <div className="cox-card">
          <span className="section-number">COX-1 · CONSTITUTIVA</span>
          <h3>Manutenção fisiológica</h3>
          <p>
            Expressa na maioria dos tecidos. Participa da proteção da mucosa gástrica
            (muco, bicarbonato e fluxo sanguíneo), da agregação plaquetária (TXA₂) e
            da perfusão renal.
          </p>
          <p className="drug-note">Inibir COX-1: efeitos adversos gastrointestinais e maior tendência a sangramento.</p>
        </div>
        <div className="cox-card">
          <span className="section-number">COX-2 · INDUZÍVEL</span>
          <h3>Resposta inflamatória</h3>
          <p>
            Induzida por citocinas, lesão e outros estímulos inflamatórios, produz
            prostaglandinas ligadas a dor, febre e inflamação. Também tem expressão
            constitutiva em locais como rins e endotélio.
          </p>
          <p className="drug-note">Inibir COX-2: efeito terapêutico, mas também parte dos efeitos renais e cardiovasculares.</p>
        </div>
      </div>

      <div className="effects reveal">
        <h3>Prostanoides, efeitos e consequências da inibição</h3>
        <div className="effects-table" role="table">
          <div className="effects-row head" role="row">
            <span>Prostanoide</span>
            <span>Papel fisiológico</span>
            <span>Ao inibir a COX</span>
          </div>
          {EFFECTS.map((e) => (
            <div className="effects-row" role="row" key={e.pg}>
              <strong>{e.pg}</strong>
              <span>{e.role}</span>
              <span>{e.inhib}</span>
            </div>
          ))}
        </div>
      </div>
    </section>
  )
}

/* ==========================================================================
   COMPARADOR DE FÁRMACOS
   ========================================================================== */

function Compare() {
  const [key, setKey] = useState('ibu')
  const d = DRUGS.find((x) => x.key === key)

  return (
    <section className="section" id="comparar">
      <div className="section-heading reveal">
        <span className="section-number">05 — COMPARAR FÁRMACOS</span>
        <h2>Mesmo alvo, mecanismos diferentes.</h2>
        <p>
          Os AINEs inibem a COX de formas distintas, e os corticosteroides atuam
          por outro caminho: a expressão gênica.
        </p>
      </div>

      <div className="compare reveal">
        <div className="compare-tabs" role="tablist">
          {DRUGS.map((x) => (
            <button
              key={x.key}
              role="tab"
              aria-selected={key === x.key}
              className={key === x.key ? 'tab-btn active' : 'tab-btn'}
              onClick={() => setKey(x.key)}
            >
              {x.name}
            </button>
          ))}
        </div>

        <div className="compare-card">
          <div className="compare-head">
            <div>
              <span className="section-number">{d.cls.toUpperCase()}</span>
              <h3>{d.name}</h3>
            </div>
          </div>
          <p className="compare-mech">{d.mech}</p>

          <div className="facts">
            <div>
              <span>ALVO</span>
              <strong>{d.target}</strong>
            </div>
            <div>
              <span>TIPO DE AÇÃO</span>
              <strong>{d.rev}</strong>
            </div>
            <div>
              <span>MEIA-VIDA</span>
              <strong>{d.half}</strong>
            </div>
          </div>

          {d.sel !== null ? (
            <div className="selectivity">
              <span className="section-number">SELETIVIDADE (ILUSTRATIVA)</span>
              <div className="sel-track">
                <div className="sel-marker" style={{ left: `${d.sel * 100}%` }}></div>
              </div>
              <div className="sel-labels">
                <span>COX-1</span>
                <em>{d.selText}</em>
                <span>COX-2</span>
              </div>
            </div>
          ) : (
            <div className="selectivity">
              <span className="section-number">VIA GENÔMICA (NÃO É INIBIÇÃO DIRETA DA COX)</span>
              <div className="genomic">
                {GENOMIC.map((g, i) => (
                  <div className="genomic-step" key={g}>
                    <b>{i + 1}</b>
                    <span>{g}</span>
                  </div>
                ))}
              </div>
            </div>
          )}

          <div className="two-col">
            <div>
              <h4>Pontos de atenção</h4>
              <ul>
                {d.attention.map((a) => (
                  <li key={a}>{a}</li>
                ))}
              </ul>
            </div>
            <div>
              <h4>Cuidados de enfermagem</h4>
              <ul>
                {d.nursing.map((a) => (
                  <li key={a}>{a}</li>
                ))}
              </ul>
            </div>
          </div>
          <p className="fineprint">
            Conteúdo educacional. Não substitui bula, prescrição, protocolos
            institucionais nem avaliação profissional. A posição na barra de
            seletividade é qualitativa e varia conforme o método de ensaio.
          </p>
        </div>
      </div>
    </section>
  )
}

/* ==========================================================================
   QUIZ
   ========================================================================== */

function Quiz() {
  const [i, setI] = useState(0)
  const [picked, setPicked] = useState(null)
  const [score, setScore] = useState(0)
  const [finished, setFinished] = useState(false)

  const q = QUIZ[i]

  const choose = (idx) => {
    if (picked !== null) return
    setPicked(idx)
    if (idx === q.answer) setScore((s) => s + 1)
  }

  const next = () => {
    if (i + 1 >= QUIZ.length) {
      setFinished(true)
    } else {
      setI(i + 1)
      setPicked(null)
    }
  }

  const restart = () => {
    setI(0)
    setPicked(null)
    setScore(0)
    setFinished(false)
  }

  const ratio = score / QUIZ.length
  const message =
    ratio >= 0.85
      ? 'Excelente domínio do percurso e do mecanismo.'
      : ratio >= 0.6
        ? 'Bom resultado. Revise as etapas em que errou e tente novamente.'
        : 'Vale revisitar as seções de farmacocinética e farmacodinâmica e refazer o quiz.'

  return (
    <section className="section" id="quiz">
      <div className="section-heading reveal">
        <span className="section-number">06 — FIXAÇÃO</span>
        <h2>Teste o que você aprendeu.</h2>
        <p>Nove perguntas sobre ADME, mecanismo de ação e cuidados de enfermagem.</p>
      </div>

      <div className="quiz-card reveal">
        {!finished ? (
          <>
            <div className="quiz-top">
              <span className="section-number">
                PERGUNTA {i + 1} DE {QUIZ.length}
              </span>
              <span className="quiz-score">Acertos: {score}</span>
            </div>
            <div className="quiz-progress">
              <div style={{ width: `${(i / QUIZ.length) * 100}%` }}></div>
            </div>
            <h3>{q.q}</h3>
            <div className="quiz-options">
              {q.options.map((o, idx) => {
                let cls = 'quiz-option'
                if (picked !== null) {
                  if (idx === q.answer) cls += ' correct'
                  else if (idx === picked) cls += ' wrong'
                }
                return (
                  <button key={o} className={cls} onClick={() => choose(idx)} disabled={picked !== null}>
                    <b>{String.fromCharCode(65 + idx)}</b>
                    <span>{o}</span>
                  </button>
                )
              })}
            </div>
            {picked !== null && (
              <div className="quiz-feedback" aria-live="polite">
                <strong>{picked === q.answer ? 'Correto.' : 'Não foi dessa vez.'}</strong> {q.why}
                <div>
                  <button className="ctrl-btn primary" onClick={next}>
                    {i + 1 >= QUIZ.length ? 'Ver resultado' : 'Próxima pergunta ▶'}
                  </button>
                </div>
              </div>
            )}
          </>
        ) : (
          <div className="quiz-result">
            <span className="section-number">RESULTADO</span>
            <h3>
              {score} de {QUIZ.length} acertos
            </h3>
            <p>{message}</p>
            <button className="ctrl-btn primary" onClick={restart}>
              ↺ Refazer o quiz
            </button>
          </div>
        )}
      </div>
    </section>
  )
}

/* ==========================================================================
   REFERÊNCIAS
   ========================================================================== */

function References() {
  return (
    <section className="section" id="referencias">
      <div className="section-heading reveal">
        <span className="section-number">07 — REFERÊNCIAS</span>
        <h2>Onde validar o conteúdo.</h2>
        <p>
          As fontes abaixo são indicações de consulta para a validação acadêmica.
          Os textos do site ainda precisam ser confrontados com as edições
          efetivamente disponíveis ao estudante.
        </p>
      </div>
      <ul className="refs reveal">
        {REFS.map((r, i) => (
          <li key={r}>
            <span>{String(i + 1).padStart(2, '0')}</span>
            {r}
          </li>
        ))}
      </ul>
      <p className="fineprint reveal">
        As ilustrações anatômicas são esquemáticas e servem para orientação
        espacial. Antes do uso como material didático definitivo, devem ser
        revisadas com atlas de anatomia e bibliografia farmacológica.
      </p>
    </section>
  )
}

/* ==========================================================================
   APLICAÇÃO
   ========================================================================== */

function App() {
  const [jump, setJump] = useState(null)
  useReveal()

  const goStage = (stage) => {
    setJump({ stage, id: Date.now() })
    const el = document.getElementById('farmacocinetica')
    if (el) el.scrollIntoView({ behavior: 'smooth', block: 'start' })
  }

  const journey = [
    { n: '01', t: 'Administração oral', p: 'O medicamento entra no organismo pela boca.', go: () => goStage(0) },
    { n: '02', t: 'Absorção', p: 'O fármaco atravessa o trato gastrointestinal.', go: () => goStage(3) },
    { n: '03', t: 'Distribuição', p: 'O medicamento alcança a circulação e os tecidos.', go: () => goStage(6) },
    {
      n: '04',
      t: 'Ação molecular',
      p: 'O fármaco interage com seus alvos farmacológicos.',
      go: () => {
        const el = document.getElementById('farmacodinamica')
        if (el) el.scrollIntoView({ behavior: 'smooth', block: 'start' })
      }
    }
  ]

  return (
    <main className="app">
      <Navbar />

      <section className="hero" id="inicio">
        <div className="hero-content">
          <div className="eyebrow">
            <span className="pulse"></span>
            CIÊNCIA · FARMACOLOGIA · FISIOLOGIA
          </div>
          <h1>
            A ciência por trás
            <br />
            da <span>inflamação.</span>
          </h1>
          <p>
            Explore o caminho dos medicamentos no organismo e descubra como eles
            interferem nos mecanismos bioquímicos da resposta inflamatória.
          </p>
          <div className="hero-actions">
            <a className="start-button" href="#farmacocinetica">
              EXPLORAR JORNADA <span>↗</span>
            </a>
            <a className="ghost-button" href="#quiz">
              IR AO QUIZ
            </a>
          </div>
          <div className="hero-meta">
            <span>01 / 07</span>
            <span>UMA EXPERIÊNCIA DE APRENDIZAGEM INTERATIVA</span>
          </div>
        </div>
        <div className="hero-visual">
          <div className="visual-ring ring-one"></div>
          <div className="visual-ring ring-two"></div>
          <div className="visual-core">
            <span>COX</span>
          </div>
          <div className="visual-label label-top">
            <span>01</span> RESPOSTA INFLAMATÓRIA
          </div>
          <div className="visual-label label-bottom">
            <span>02</span> MECANISMOS MOLECULARES
          </div>
        </div>
      </section>

      <section className="journey" id="roteiro">
        <div className="journey-heading reveal">
          <span className="section-number">01 — ROTEIRO</span>
          <h2>Do organismo à molécula.</h2>
          <p>
            Acompanhe o percurso de um medicamento desde sua administração até seu
            local de ação. Clique em uma etapa para ir direto a ela.
          </p>
        </div>
        {journey.map((j) => (
          <button className="journey-card reveal" key={j.n} onClick={j.go}>
            <span>{j.n}</span>
            <h3>{j.t}</h3>
            <p>{j.p}</p>
            <em>Ver etapa ↗</em>
          </button>
        ))}
      </section>

      <Pharmacokinetics jump={jump} />
      <ScaleZoom />
      <Pharmacodynamics />
      <Compare />
      <Quiz />
      <References />

      <footer id="sobre">
        <span>INFLAMMA © 2026</span>
        <span>PLATAFORMA EDUCACIONAL DE FARMACOLOGIA · CONTEÚDO EDUCACIONAL</span>
      </footer>
    </main>
  )
}

export default App
