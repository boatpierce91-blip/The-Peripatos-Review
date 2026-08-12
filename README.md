# The-Peripatos-Review
import React, { useState, useEffect, useRef } from "react";
import { Brain, Leaf, Globe, Link2, Menu, X, ArrowRight, ArrowLeft, Quote, Sparkles, Search } from "lucide-react";

/* ============================================================
   THE PERIPATOS REVIEW — single-file app with real navigation.
   No react-router (not available in this environment), so routing
   is done with a small hand-rolled hash router: page state synced
   to window.location.hash, so links, back/forward, and refresh
   all behave like real pages. The router also understands a
   second hash segment as a "param" (e.g. #article/some-slug),
   used to open a specific article or issue.

   ------------------------------------------------------------
   CONTENT ARCHITECTURE — read this before touching anything else
   ------------------------------------------------------------
   Everything publishable lives in the CONTENT section right below
   this comment: the ARTICLES array, the ISSUES array, and the
   EDITORS array. Every page below (Articles, an Issue, a single
   Article, the homepage "latest issue" block, even the nav label
   that says "Issue 001") reads from that data instead of having
   its own hardcoded copy.

   To publish a new article:
     1. Add an object to ARTICLES (copy an existing one as a
        template — see the field-by-field notes above the array).
     2. Set status: "published" and fill in `body`.
     That's it — it will automatically show up in the Articles
     archive, on its Issue page, and get its own article page at
     #article/<slug>. No page component needs to change.

   To open a new issue:
     1. Add an object to ISSUES.
     2. Point new ARTICLES at it via their `issue` field.
     The homepage, nav, and Issue page pick it up automatically —
     the nav always labels the "Issue" link with the most recent
     issue in the array.
   ============================================================ */

const AREAS = [
  {
    key: "mind",
    numeral: "I",
    name: "Mind",
    icon: Brain,
    grad: "from-indigo-600 to-violet-600",
    solid: "bg-indigo-600",
    tint: "bg-indigo-50",
    text: "text-indigo-700",
    desc: "Philosophy and cognitive science — consciousness, free will, personal identity, memory, and the philosophy of artificial minds.",
  },
  {
    key: "life",
    numeral: "II",
    name: "Life",
    icon: Leaf,
    grad: "from-emerald-600 to-teal-600",
    solid: "bg-emerald-600",
    tint: "bg-emerald-50",
    text: "text-emerald-700",
    desc: "Biology and the systems that make it up — evolution, genetics, animal behavior, adaptation, and the origins of life.",
  },
  {
    key: "world",
    numeral: "III",
    name: "World",
    icon: Globe,
    grad: "from-teal-600 to-cyan-600",
    solid: "bg-teal-600",
    tint: "bg-teal-50",
    text: "text-teal-700",
    desc: "Environmental science and our relationship with nature — ecology, climate, conservation, and environmental ethics.",
  },
  {
    key: "connections",
    numeral: "IV",
    name: "Connections",
    icon: Link2,
    grad: "from-orange-600 to-amber-500",
    solid: "bg-orange-600",
    tint: "bg-orange-50",
    text: "text-orange-700",
    desc: "Writing that crosses the lines between the three — where philosophy meets biology, or ecology meets ethics.",
  },
];

const TYPES = ["Essay", "Research Review", "Interdisciplinary", "Book/Paper Review"];

/* ============================================================
   CONTENT — the only section you should need to edit to publish
   new work. Pages below read from this data; they don't hold
   their own copies of articles or issues.
   ============================================================ */

/*
  ARTICLES — field notes
  ----------------------------------------------------------------
  slug        unique, url-safe id, e.g. "where-does-the-mind-begin"
              (used in the address bar as #article/<slug>)
  title       headline
  dek         one-sentence summary shown on cards and the article header
  area        one of: "mind" | "life" | "world" | "connections"
  type        one of TYPES above
  status      "published" — live everywhere: archive, issue page,
                             homepage, and its own article page
              "planned"   — shown only as a proposed topic on its
                             issue's page, not in the archive, and
                             has no article page yet
  issue       slug of the issue this belongs to (see ISSUES below)
  author      display name — leave "" until assigned
  authorBio   optional one-line bio shown at the foot of the article
  date        "YYYY-MM-DD" — only needed once published
  crossing    short "Field / Field" tag shown on proposed-topic cards
  tags        lowercase keywords the search box matches against
  body        array of paragraph strings. Leave as [] until the
              piece is written — the article page shows a
              "manuscript in progress" state instead of erroring.

  Example of a finished, published article (for reference — not
  live, since it's commented out):

  {
    slug: "why-memory-lies",
    title: "Why Memory Lies to You",
    dek: "Memory doesn't record the past — it reconstructs it, and that reconstruction is where the philosophy gets interesting.",
    area: "mind",
    type: "Essay",
    status: "published",
    issue: "001",
    author: "Jal Patidar",
    authorBio: "Jal is a high-school student interested in philosophy and cognitive science.",
    date: "2026-09-14",
    crossing: "Philosophy / Cognitive Science",
    tags: ["memory", "consciousness", "identity"],
    body: [
      "First paragraph of the piece...",
      "Second paragraph...",
    ],
  },
*/
const ARTICLES = [
  {
    slug: "where-does-the-mind-begin",
    title: "Where Does the Mind Begin?",
    dek: "A look at where philosophers and cognitive scientists draw the line between brain and mind.",
    area: "mind",
    type: "Essay",
    status: "planned",
    issue: "001",
    author: "",
    authorBio: "",
    date: "",
    crossing: "Philosophy / Cognitive Science",
    tags: ["consciousness", "mind", "cognitive science"],
    body: [],
  },
  {
    slug: "can-a-plant-be-intelligent",
    title: "Can a Plant Be Intelligent?",
    dek: "Plants sense, signal, and respond to their environment — does that count as a kind of intelligence?",
    area: "connections",
    type: "Interdisciplinary",
    status: "planned",
    issue: "001",
    author: "",
    authorBio: "",
    date: "",
    crossing: "Biology / Philosophy",
    tags: ["plants", "intelligence", "biology"],
    body: [],
  },
  {
    slug: "are-humans-separate-from-nature",
    title: "Are Humans Separate From Nature?",
    dek: "Environmental philosophy's oldest question, asked again for an era of climate change.",
    area: "connections",
    type: "Interdisciplinary",
    status: "planned",
    issue: "001",
    author: "",
    authorBio: "",
    date: "",
    crossing: "Environmental Philosophy",
    tags: ["nature", "environment", "ethics"],
    body: [],
  },
  {
    slug: "what-makes-something-alive",
    title: "What Makes Something Alive?",
    dek: "From viruses to von Neumann machines, the boundary of 'life' is blurrier than it looks.",
    area: "life",
    type: "Research Review",
    status: "planned",
    issue: "001",
    author: "",
    authorBio: "",
    date: "",
    crossing: "Biology / Philosophy",
    tags: ["life", "biology", "definitions"],
    body: [],
  },
  {
    slug: "could-an-artificial-system-ever-be-conscious",
    title: "Could an Artificial System Ever Be Conscious?",
    dek: "What philosophy of mind can and can't tell us about machine consciousness.",
    area: "connections",
    type: "Interdisciplinary",
    status: "planned",
    issue: "001",
    author: "",
    authorBio: "",
    date: "",
    crossing: "Cognitive Science / Philosophy",
    tags: ["ai", "consciousness", "cognitive science"],
    body: [],
  },
  {
    slug: "is-an-ecosystem-an-individual",
    title: "Is an Ecosystem an Individual?",
    dek: "Ecology keeps finding systems that act like organisms. Philosophy asks what that means.",
    area: "connections",
    type: "Interdisciplinary",
    status: "planned",
    issue: "001",
    author: "",
    authorBio: "",
    date: "",
    crossing: "Ecology / Philosophy",
    tags: ["ecology", "systems", "philosophy"],
    body: [],
  },
];

/*
  ISSUES — field notes
  ----------------------------------------------------------------
  slug     unique id, e.g. "001" (used as #issue/<slug>)
  number   display number, e.g. "001"
  title    issue title
  theme    one- or two-sentence description shown in the issue header
  status   "upcoming" | "published"

  List issues oldest-first — the nav and homepage always feature
  the last item in this array as the current issue.
*/
const ISSUES = [
  {
    slug: "001",
    number: "001",
    title: "Where Do We Draw the Line?",
    theme:
      "Our founding issue asks where the boundaries really are — between mind and brain, life and non-life, humanity and nature. It's the first issue built to represent both sides of the Review from day one: philosophy and cognitive science alongside biology and environmental science.",
    status: "upcoming",
  },
];

const EDITORS = [
  {
    name: "Jal Patidar",
    initials: "JP",
    grad: "from-indigo-600 to-violet-600",
    bio: "Jal is a high-school student interested in philosophy and cognitive science, particularly questions surrounding consciousness, knowledge, identity, and the relationship between the mind and the world.",
  },
  {
    name: "Kushagra Saraswat",
    initials: "KS",
    grad: "from-teal-600 to-emerald-600",
    bio: "Kushagra is a high-school student interested in biology and environmental science, with a particular interest in living systems, ecology, and the relationship between life and its environment.",
  },
];

/* ---------------- content helpers ---------------- */
function getPublishedArticles() {
  return ARTICLES.filter((a) => a.status === "published").sort((a, b) =>
    (b.date || "").localeCompare(a.date || "")
  );
}
function getPlannedArticles(issueSlug) {
  return ARTICLES.filter((a) => a.issue === issueSlug && a.status === "planned");
}
function getArticlesByIssue(issueSlug) {
  return getPublishedArticles().filter((a) => a.issue === issueSlug);
}
function getArticleBySlug(slug) {
  return ARTICLES.find((a) => a.slug === slug);
}
function getIssueBySlug(slug) {
  return ISSUES.find((i) => i.slug === slug);
}
function getLatestIssue() {
  return ISSUES.length ? ISSUES[ISSUES.length - 1] : null;
}
function getAreaByKey(key) {
  return AREAS.find((a) => a.key === key);
}
function formatDate(iso) {
  if (!iso) return "";
  const d = new Date(`${iso}T00:00:00`);
  if (Number.isNaN(d.getTime())) return "";
  return d.toLocaleDateString("en-US", { month: "short", year: "numeric" });
}

/* ---------------- tiny hash router helpers ---------------- */
// Hash format: #<page> or #<page>/<param>, e.g. #article/where-does-the-mind-begin
function parseHash() {
  const raw = window.location.hash.replace("#", "");
  if (!raw) return { page: "home", param: null };
  const [page, ...rest] = raw.split("/");
  return { page: page || "home", param: rest.length ? rest.join("/") : null };
}

export default function PeripatosSite() {
  const [page, setPage] = useState("home");
  const [param, setParam] = useState(null);
  const [menuOpen, setMenuOpen] = useState(false);
  const [articleAreaFilter, setArticleAreaFilter] = useState("all");
  const topRef = useRef(null);

  // sync page <-> hash so links/back-forward/refresh all work
  useEffect(() => {
    const sync = () => {
      const { page: p, param: pr } = parseHash();
      setPage(p);
      setParam(pr);
    };
    sync();
    window.addEventListener("hashchange", sync);
    document.documentElement.style.scrollBehavior = "smooth";
    return () => window.removeEventListener("hashchange", sync);
  }, []);

  useEffect(() => {
    if (topRef.current) topRef.current.scrollIntoView({ behavior: "instant" in window ? "instant" : "auto" });
    window.scrollTo(0, 0);
  }, [page, param]);

  function navigate(nextPage, opts = {}) {
    setMenuOpen(false);
    if (opts.area) setArticleAreaFilter(opts.area);
    const hash = opts.slug ? `${nextPage}/${opts.slug}` : nextPage;
    window.location.hash = hash;
    setPage(nextPage);
    setParam(opts.slug || null);
  }

  const latestIssue = getLatestIssue();

  return (
    <div className="font-body bg-stone-50 text-stone-900 antialiased" ref={topRef}>
      <FontsAndKeyframes />
      <div className="h-1.5 w-full" style={{ background: "linear-gradient(90deg, #4f46e5, #059669, #0d9488, #ea580c)" }} />

      <Nav page={page} navigate={navigate} menuOpen={menuOpen} setMenuOpen={setMenuOpen} latestIssue={latestIssue} />

      <main>
        {page === "home" && <HomePage navigate={navigate} />}
        {page === "articles" && (
          <ArticlesPage areaFilter={articleAreaFilter} setAreaFilter={setArticleAreaFilter} navigate={navigate} />
        )}
        {page === "article" && <ArticlePage slug={param} navigate={navigate} />}
        {page === "issue" && <IssuePage slug={param} navigate={navigate} />}
        {page === "about" && <AboutPage navigate={navigate} />}
        {page === "editors" && <EditorsPage navigate={navigate} />}
        {page === "submit" && <SubmitPage navigate={navigate} />}
        {page === "contact" && <ContactPage navigate={navigate} />}
      </main>

      <Footer navigate={navigate} />
    </div>
  );
}

/* ============================================================
   NAV
   ============================================================ */
function Nav({ page, navigate, menuOpen, setMenuOpen, latestIssue }) {
  const navLinks = [
    { page: "home", label: "Home" },
    { page: "articles", label: "Articles" },
    { page: "issue", label: latestIssue ? `Issue ${latestIssue.number}` : "Issues" },
    { page: "about", label: "About" },
    { page: "editors", label: "Editors" },
    { page: "contact", label: "Contact" },
  ];

  return (
    <header className="sticky top-0 z-50 border-b border-stone-200 bg-stone-50/85 backdrop-blur-md">
      <div className="mx-auto flex max-w-6xl items-center justify-between px-5 py-3 sm:px-8">
        <button onClick={() => navigate("home")} className="flex flex-col items-start leading-none">
          <span className="font-display text-lg italic tracking-tight text-stone-900">The Peripatos Review</span>
          <span className="font-mono-label mt-1 text-[10px] tracking-[0.25em] text-stone-500">MIND · LIFE · WORLD</span>
        </button>

        <nav className="hidden items-center gap-6 md:flex">
          {navLinks.map((l) => (
            <button
              key={l.page}
              onClick={() => navigate(l.page)}
              className={`font-mono-label text-xs uppercase tracking-wider transition ${
                page === l.page ? "text-stone-950 border-b-2 border-orange-500 pb-0.5" : "text-stone-600 hover:text-stone-950"
              }`}
            >
              {l.label}
            </button>
          ))}
          <button
            onClick={() => navigate("submit")}
            className="rounded-full bg-gradient-to-r from-indigo-600 to-orange-500 px-5 py-2.5 text-xs font-semibold uppercase tracking-wider text-white shadow-lg shadow-indigo-900/20 transition hover:scale-105"
          >
            Submit
          </button>
        </nav>

        <button className="rounded-md border border-stone-300 p-2 md:hidden" onClick={() => setMenuOpen((v) => !v)} aria-label="Toggle menu">
          {menuOpen ? <X size={18} /> : <Menu size={18} />}
        </button>
      </div>

      {menuOpen && (
        <div className="border-t border-stone-200 bg-stone-50 px-5 pb-5 md:hidden">
          <div className="flex flex-col gap-1 pt-3">
            {navLinks.map((l) => (
              <button
                key={l.page}
                onClick={() => navigate(l.page)}
                className={`rounded-md px-2 py-2.5 text-left font-mono-label text-xs uppercase tracking-wider ${
                  page === l.page ? "bg-stone-100 text-stone-950" : "text-stone-700 hover:bg-stone-100"
                }`}
              >
                {l.label}
              </button>
            ))}
            <button
              onClick={() => navigate("submit")}
              className="mt-2 rounded-full bg-gradient-to-r from-indigo-600 to-orange-500 px-5 py-2.5 text-center text-xs font-semibold uppercase tracking-wider text-white"
            >
              Submit
            </button>
          </div>
        </div>
      )}
    </header>
  );
}

/* ============================================================
   HOME
   ============================================================ */
function HomePage({ navigate }) {
  const issue = getLatestIssue();
  const foundingNumber = ISSUES.length ? ISSUES[0].number : "—";

  return (
    <>
      <section className="relative overflow-hidden bg-stone-950 text-stone-50">
        <div className="pointer-events-none absolute -left-24 -top-24 h-96 w-96 rounded-full bg-indigo-600/40 blur-[100px]" />
        <div className="pointer-events-none absolute right-[-6rem] top-10 h-80 w-80 rounded-full bg-emerald-500/30 blur-[100px]" />
        <div className="pointer-events-none absolute bottom-[-8rem] left-1/3 h-96 w-96 rounded-full bg-teal-500/25 blur-[110px]" />
        <div className="pointer-events-none absolute bottom-[-4rem] right-10 h-72 w-72 rounded-full bg-orange-500/25 blur-[100px]" />

        <div className="relative mx-auto grid max-w-6xl grid-cols-1 gap-12 px-5 pb-16 pt-16 sm:px-8 sm:pt-24 lg:grid-cols-[1.15fr_0.85fr] lg:items-center lg:pb-24">
          <div>
            <p className="font-mono-label inline-flex items-center gap-2 rounded-full border border-white/15 bg-white/5 px-3 py-1.5 text-[11px] uppercase tracking-[0.2em] text-orange-300">
              <Sparkles size={12} /> Est. 2026 · Independent student publication
            </p>

            <h1 className="font-display mt-6 text-[2.6rem] font-medium leading-[1.05] tracking-tight sm:text-6xl lg:text-[4.2rem]">
              Questions worth{" "}
              <em className="bg-gradient-to-r from-indigo-400 via-emerald-300 to-orange-300 bg-clip-text italic text-transparent">
                wandering
              </em>{" "}
              with.
            </h1>

            <p className="font-mono-label mt-6 text-sm uppercase tracking-[0.3em] text-emerald-300">Mind. Life. World.</p>

            <p className="mt-4 max-w-xl text-lg leading-relaxed text-stone-300">
              A student-run publication for curious young thinkers — essays, research reviews, interdisciplinary
              writing, and critical reviews exploring the mind, life, and world around us.
            </p>

            <p className="font-display mt-4 text-xl italic text-stone-100">Good questions don't have an age requirement.</p>

            <div className="mt-9 flex flex-wrap gap-3">
              <button
                onClick={() => navigate("articles")}
                className="inline-flex items-center gap-2 rounded-full bg-gradient-to-r from-orange-500 to-amber-400 px-6 py-3.5 text-sm font-semibold text-stone-950 shadow-xl shadow-orange-900/30 transition hover:scale-105"
              >
                Explore the Review <ArrowRight size={16} />
              </button>
              <button
                onClick={() => navigate("issue", issue ? { slug: issue.slug } : {})}
                className="inline-flex items-center gap-2 rounded-full bg-gradient-to-r from-indigo-500 to-violet-500 px-6 py-3.5 text-sm font-semibold text-white shadow-xl shadow-indigo-900/30 transition hover:scale-105"
              >
                Preview Issue {issue ? issue.number : ""}
              </button>
              <button
                onClick
