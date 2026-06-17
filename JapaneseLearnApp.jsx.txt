import { useState, useEffect, useRef, useCallback } from "react";

// ─── DATA ────────────────────────────────────────────────────────────────────

const HIRAGANA = [
  {char:"あ",romaji:"a"},{char:"い",romaji:"i"},{char:"う",romaji:"u"},{char:"え",romaji:"e"},{char:"お",romaji:"o"},
  {char:"か",romaji:"ka"},{char:"き",romaji:"ki"},{char:"く",romaji:"ku"},{char:"け",romaji:"ke"},{char:"こ",romaji:"ko"},
  {char:"さ",romaji:"sa"},{char:"し",romaji:"shi"},{char:"す",romaji:"su"},{char:"せ",romaji:"se"},{char:"そ",romaji:"so"},
  {char:"た",romaji:"ta"},{char:"ち",romaji:"chi"},{char:"つ",romaji:"tsu"},{char:"て",romaji:"te"},{char:"と",romaji:"to"},
  {char:"な",romaji:"na"},{char:"に",romaji:"ni"},{char:"ぬ",romaji:"nu"},{char:"ね",romaji:"ne"},{char:"の",romaji:"no"},
  {char:"は",romaji:"ha"},{char:"ひ",romaji:"hi"},{char:"ふ",romaji:"fu"},{char:"へ",romaji:"he"},{char:"ほ",romaji:"ho"},
  {char:"ま",romaji:"ma"},{char:"み",romaji:"mi"},{char:"む",romaji:"mu"},{char:"め",romaji:"me"},{char:"も",romaji:"mo"},
  {char:"や",romaji:"ya"},{char:"ゆ",romaji:"yu"},{char:"よ",romaji:"yo"},
  {char:"ら",romaji:"ra"},{char:"り",romaji:"ri"},{char:"る",romaji:"ru"},{char:"れ",romaji:"re"},{char:"ろ",romaji:"ro"},
  {char:"わ",romaji:"wa"},{char:"を",romaji:"wo"},{char:"ん",romaji:"n"},
];

const KATAKANA = [
  {char:"ア",romaji:"a"},{char:"イ",romaji:"i"},{char:"ウ",romaji:"u"},{char:"エ",romaji:"e"},{char:"オ",romaji:"o"},
  {char:"カ",romaji:"ka"},{char:"キ",romaji:"ki"},{char:"ク",romaji:"ku"},{char:"ケ",romaji:"ke"},{char:"コ",romaji:"ko"},
  {char:"サ",romaji:"sa"},{char:"シ",romaji:"shi"},{char:"ス",romaji:"su"},{char:"セ",romaji:"se"},{char:"ソ",romaji:"so"},
  {char:"タ",romaji:"ta"},{char:"チ",romaji:"chi"},{char:"ツ",romaji:"tsu"},{char:"テ",romaji:"te"},{char:"ト",romaji:"to"},
  {char:"ナ",romaji:"na"},{char:"ニ",romaji:"ni"},{char:"ヌ",romaji:"nu"},{char:"ネ",romaji:"ne"},{char:"ノ",romaji:"no"},
  {char:"ハ",romaji:"ha"},{char:"ヒ",romaji:"hi"},{char:"フ",romaji:"fu"},{char:"ヘ",romaji:"he"},{char:"ホ",romaji:"ho"},
  {char:"マ",romaji:"ma"},{char:"ミ",romaji:"mi"},{char:"ム",romaji:"mu"},{char:"メ",romaji:"me"},{char:"モ",romaji:"mo"},
  {char:"ヤ",romaji:"ya"},{char:"ユ",romaji:"yu"},{char:"ヨ",romaji:"yo"},
  {char:"ラ",romaji:"ra"},{char:"リ",romaji:"ri"},{char:"ル",romaji:"ru"},{char:"レ",romaji:"re"},{char:"ロ",romaji:"ro"},
  {char:"ワ",romaji:"wa"},{char:"ヲ",romaji:"wo"},{char:"ン",romaji:"n"},
];

const VOCAB = {
  greetings: {
    label:"Greetings", icon:"👋", color:"#E84393",
    words:[
      {jp:"こんにちは",romaji:"Konnichiwa",en:"Hello / Good afternoon",ex:"こんにちは、田中さん！",exEn:"Hello, Tanaka-san!"},
      {jp:"おはようございます",romaji:"Ohayou gozaimasu",en:"Good morning (formal)",ex:"おはようございます！",exEn:"Good morning!"},
      {jp:"こんばんは",romaji:"Konbanwa",en:"Good evening",ex:"こんばんは、元気ですか？",exEn:"Good evening, how are you?"},
      {jp:"おやすみなさい",romaji:"Oyasumi nasai",en:"Good night",ex:"おやすみなさい！",exEn:"Good night!"},
      {jp:"さようなら",romaji:"Sayounara",en:"Goodbye",ex:"さようなら、また明日！",exEn:"Goodbye, see you tomorrow!"},
      {jp:"はじめまして",romaji:"Hajimemashite",en:"Nice to meet you",ex:"はじめまして、ラジです。",exEn:"Nice to meet you, I'm Raj."},
      {jp:"よろしくおねがいします",romaji:"Yoroshiku onegaishimasu",en:"Please treat me well",ex:"よろしくおねがいします！",exEn:"I look forward to working with you!"},
      {jp:"ありがとうございます",romaji:"Arigatou gozaimasu",en:"Thank you (formal)",ex:"ありがとうございます！",exEn:"Thank you very much!"},
      {jp:"どういたしまして",romaji:"Dou itashimashite",en:"You're welcome",ex:"どういたしまして！",exEn:"You're welcome!"},
      {jp:"すみません",romaji:"Sumimasen",en:"Excuse me / Sorry",ex:"すみません、トイレはどこですか？",exEn:"Excuse me, where is the restroom?"},
    ]
  },
  food: {
    label:"Food & Drink", icon:"🍜", color:"#F59E0B",
    words:[
      {jp:"ごはん",romaji:"Gohan",en:"Rice / Meal",ex:"ごはんを食べます。",exEn:"I eat rice."},
      {jp:"みず",romaji:"Mizu",en:"Water",ex:"みずをください。",exEn:"Please give me water."},
      {jp:"おちゃ",romaji:"Ocha",en:"Green tea",ex:"おちゃがすきです。",exEn:"I like green tea."},
      {jp:"すし",romaji:"Sushi",en:"Sushi",ex:"すしがたべたい！",exEn:"I want to eat sushi!"},
      {jp:"ラーメン",romaji:"Raamen",en:"Ramen",ex:"ラーメンはおいしいです。",exEn:"Ramen is delicious."},
      {jp:"さかな",romaji:"Sakana",en:"Fish",ex:"さかなをたべます。",exEn:"I eat fish."},
      {jp:"にく",romaji:"Niku",en:"Meat",ex:"にくがすきですか？",exEn:"Do you like meat?"},
      {jp:"やさい",romaji:"Yasai",en:"Vegetables",ex:"やさいはからだにいいです。",exEn:"Vegetables are good for the body."},
      {jp:"くだもの",romaji:"Kudamono",en:"Fruit",ex:"くだものをたべましょう。",exEn:"Let's eat some fruit."},
      {jp:"コーヒー",romaji:"Koohii",en:"Coffee",ex:"コーヒーをのみます。",exEn:"I drink coffee."},
    ]
  },
  numbers: {
    label:"Numbers", icon:"🔢", color:"#8B5CF6",
    words:[
      {jp:"いち",romaji:"ichi",en:"1",ex:"いちまい",exEn:"one sheet"},
      {jp:"に",romaji:"ni",en:"2",ex:"にほん",exEn:"two (long things)"},
      {jp:"さん",romaji:"san",en:"3",ex:"さんびき",exEn:"three animals"},
      {jp:"し・よん",romaji:"shi / yon",en:"4",ex:"よんじゅう",exEn:"forty"},
      {jp:"ご",romaji:"go",en:"5",ex:"ごじ",exEn:"five o'clock"},
      {jp:"ろく",romaji:"roku",en:"6",ex:"ろくがつ",exEn:"June"},
      {jp:"しち・なな",romaji:"shichi / nana",en:"7",ex:"なながつ",exEn:"July"},
      {jp:"はち",romaji:"hachi",en:"8",ex:"はちじ",exEn:"eight o'clock"},
      {jp:"く・きゅう",romaji:"ku / kyuu",en:"9",ex:"きゅうじゅう",exEn:"ninety"},
      {jp:"じゅう",romaji:"juu",en:"10",ex:"じゅういち",exEn:"eleven"},
      {jp:"ひゃく",romaji:"hyaku",en:"100",ex:"ひゃくえん",exEn:"100 yen"},
      {jp:"せん",romaji:"sen",en:"1000",ex:"いっせんえん",exEn:"1000 yen"},
    ]
  },
  colors: {
    label:"Colors", icon:"🎨", color:"#EC4899",
    words:[
      {jp:"あか",romaji:"aka",en:"Red",ex:"あかいりんご",exEn:"red apple"},
      {jp:"あお",romaji:"ao",en:"Blue",ex:"あおいそら",exEn:"blue sky"},
      {jp:"きいろ",romaji:"kiiro",en:"Yellow",ex:"きいろいはな",exEn:"yellow flower"},
      {jp:"みどり",romaji:"midori",en:"Green",ex:"みどりのき",exEn:"green tree"},
      {jp:"しろ",romaji:"shiro",en:"White",ex:"しろいゆき",exEn:"white snow"},
      {jp:"くろ",romaji:"kuro",en:"Black",ex:"くろいねこ",exEn:"black cat"},
      {jp:"ももいろ",romaji:"momoiro",en:"Pink",ex:"ももいろのはな",exEn:"pink flower"},
      {jp:"むらさき",romaji:"murasaki",en:"Purple",ex:"むらさきのぶどう",exEn:"purple grapes"},
    ]
  },
  animals: {
    label:"Animals", icon:"🐾", color:"#10B981",
    words:[
      {jp:"いぬ",romaji:"inu",en:"Dog",ex:"いぬがすきです。",exEn:"I like dogs."},
      {jp:"ねこ",romaji:"neko",en:"Cat",ex:"ねこがいます。",exEn:"There is a cat."},
      {jp:"さかな",romaji:"sakana",en:"Fish",ex:"さかながおよぎます。",exEn:"The fish swims."},
      {jp:"とり",romaji:"tori",en:"Bird",ex:"とりがとびます。",exEn:"The bird flies."},
      {jp:"うさぎ",romaji:"usagi",en:"Rabbit",ex:"うさぎがはしります。",exEn:"The rabbit runs."},
      {jp:"くま",romaji:"kuma",en:"Bear",ex:"くまはおおきいです。",exEn:"The bear is big."},
      {jp:"さる",romaji:"saru",en:"Monkey",ex:"さるはかしこいです。",exEn:"Monkeys are smart."},
      {jp:"ぞう",romaji:"zou",en:"Elephant",ex:"ぞうははなながいです。",exEn:"Elephants have long trunks."},
    ]
  },
  family: {
    label:"Family", icon:"👨‍👩‍👧", color:"#6366F1",
    words:[
      {jp:"おとうさん",romaji:"otousan",en:"Father",ex:"おとうさんはいしゃです。",exEn:"My father is a doctor."},
      {jp:"おかあさん",romaji:"okaasan",en:"Mother",ex:"おかあさんはせんせいです。",exEn:"My mother is a teacher."},
      {jp:"あに",romaji:"ani",en:"Older brother",ex:"あにはだいがくせいです。",exEn:"My older brother is a university student."},
      {jp:"あね",romaji:"ane",en:"Older sister",ex:"あねはかいしゃいんです。",exEn:"My older sister is an office worker."},
      {jp:"いもうと",romaji:"imouto",en:"Younger sister",ex:"いもうとはこどもです。",exEn:"My younger sister is a child."},
      {jp:"おとうと",romaji:"otouto",en:"Younger brother",ex:"おとうとはがくせいです。",exEn:"My younger brother is a student."},
      {jp:"そふ",romaji:"sofu",en:"Grandfather",ex:"そふはなんさいですか？",exEn:"How old is your grandfather?"},
      {jp:"そぼ",romaji:"sobo",en:"Grandmother",ex:"そぼはやさしいです。",exEn:"My grandmother is kind."},
    ]
  },
};

const KANJI = [
  {kanji:"日",romaji:"nichi/hi",on:"ニチ",kun:"ひ",en:"Sun / Day",stroke:4,ex:"日本（にほん）Japan"},
  {kanji:"月",romaji:"getsu/tsuki",on:"ゲツ",kun:"つき",en:"Moon / Month",stroke:4,ex:"月曜日（げつようび）Monday"},
  {kanji:"火",romaji:"ka/hi",on:"カ",kun:"ひ",en:"Fire",stroke:4,ex:"火曜日（かようび）Tuesday"},
  {kanji:"水",romaji:"sui/mizu",on:"スイ",kun:"みず",en:"Water",stroke:4,ex:"水曜日（すいようび）Wednesday"},
  {kanji:"木",romaji:"moku/ki",on:"モク",kun:"き",en:"Tree / Wood",stroke:4,ex:"木曜日（もくようび）Thursday"},
  {kanji:"金",romaji:"kin/kane",on:"キン",kun:"かね",en:"Gold / Money",stroke:8,ex:"金曜日（きんようび）Friday"},
  {kanji:"土",romaji:"do/tsuchi",on:"ド",kun:"つち",en:"Earth / Soil",stroke:3,ex:"土曜日（どようび）Saturday"},
  {kanji:"山",romaji:"san/yama",on:"サン",kun:"やま",en:"Mountain",stroke:3,ex:"富士山（ふじさん）Mt. Fuji"},
  {kanji:"川",romaji:"sen/kawa",on:"セン",kun:"かわ",en:"River",stroke:3,ex:"川（かわ）river"},
  {kanji:"人",romaji:"jin/hito",on:"ジン",kun:"ひと",en:"Person",stroke:2,ex:"日本人（にほんじん）Japanese person"},
  {kanji:"大",romaji:"dai/oo",on:"ダイ",kun:"おお",en:"Big / Large",stroke:3,ex:"大学（だいがく）university"},
  {kanji:"小",romaji:"shou/chii",on:"ショウ",kun:"ちい",en:"Small",stroke:3,ex:"小学校（しょうがっこう）elementary school"},
  {kanji:"中",romaji:"chuu/naka",on:"チュウ",kun:"なか",en:"Middle / Inside",stroke:4,ex:"中学校（ちゅうがっこう）middle school"},
  {kanji:"上",romaji:"jou/ue",on:"ジョウ",kun:"うえ",en:"Up / Above",stroke:3,ex:"上手（じょうず）skillful"},
  {kanji:"下",romaji:"ka/shita",on:"カ",kun:"した",en:"Down / Below",stroke:3,ex:"地下（ちか）underground"},
];

const GRAMMAR = [
  {
    title:"は (wa) — Topic Marker",
    level:"N5",
    pattern:"[Topic] は [Comment] です",
    en:"Marks the topic of the sentence",
    examples:[
      {jp:"わたしは がくせい です。",en:"I am a student."},
      {jp:"これは ほん です。",en:"This is a book."},
      {jp:"かれは にほんじん です。",en:"He is Japanese."},
    ],
    tip:"は is written as 'ha' but pronounced 'wa' when used as topic marker!"
  },
  {
    title:"が (ga) — Subject Marker",
    level:"N5",
    pattern:"[Subject] が [Verb/Adjective]",
    en:"Marks the grammatical subject",
    examples:[
      {jp:"ねこ が います。",en:"There is a cat."},
      {jp:"あめ が ふります。",en:"It rains."},
      {jp:"わたし が します。",en:"I will do it."},
    ],
    tip:"Use が when introducing new info, は for known topics."
  },
  {
    title:"を (wo) — Object Marker",
    level:"N5",
    pattern:"[Object] を [Verb]",
    en:"Marks the direct object of an action",
    examples:[
      {jp:"ごはん を たべます。",en:"I eat rice."},
      {jp:"ほん を よみます。",en:"I read a book."},
      {jp:"えいご を べんきょうします。",en:"I study English."},
    ],
    tip:"を is written 'wo' but pronounced 'o' in modern Japanese."
  },
  {
    title:"に (ni) — Direction / Time",
    level:"N5",
    pattern:"[Place/Time] に",
    en:"Marks destination, location, or specific time",
    examples:[
      {jp:"がっこう に いきます。",en:"I go to school."},
      {jp:"さんじ に あいましょう。",en:"Let's meet at 3 o'clock."},
      {jp:"つくえ の うえ に あります。",en:"It's on top of the desk."},
    ],
    tip:"Think of に as 'at', 'to', or 'on' depending on context."
  },
  {
    title:"〜ます (masu) — Polite Verb",
    level:"N5",
    pattern:"[Verb stem] + ます",
    en:"Polite present/future tense",
    examples:[
      {jp:"たべます。",en:"I eat / I will eat."},
      {jp:"のみます。",en:"I drink / I will drink."},
      {jp:"みます。",en:"I watch / I will watch."},
    ],
    tip:"〜ます is the safe, polite form — use it with strangers and seniors!"
  },
  {
    title:"〜ません (masen) — Negative Polite",
    level:"N5",
    pattern:"[Verb stem] + ません",
    en:"Polite negative form",
    examples:[
      {jp:"たべません。",en:"I don't eat."},
      {jp:"わかりません。",en:"I don't understand."},
      {jp:"いきません。",en:"I don't go."},
    ],
    tip:"Just swap ます for ません to negate any polite verb!"
  },
  {
    title:"〜ですか (desu ka) — Question",
    level:"N5",
    pattern:"[Statement] + ですか？",
    en:"Turns a statement into a yes/no question",
    examples:[
      {jp:"がくせい ですか？",en:"Are you a student?"},
      {jp:"おいしい ですか？",en:"Is it delicious?"},
      {jp:"にほんじん ですか？",en:"Are you Japanese?"},
    ],
    tip:"Just add か at the end! No need to rearrange the sentence like English."
  },
  {
    title:"〜たい (tai) — Want to",
    level:"N4",
    pattern:"[Verb stem] + たい です",
    en:"Expresses desire to do something",
    examples:[
      {jp:"すし を たべたい です。",en:"I want to eat sushi."},
      {jp:"にほん に いきたい です。",en:"I want to go to Japan."},
      {jp:"にほんご を はなしたい です。",en:"I want to speak Japanese."},
    ],
    tip:"たい works like an adjective — you can say たくない for 'don't want to'!"
  },
];

const SENTENCES = [
  {jp:"わたしのなまえはラジです。",en:"My name is Raj.",romaji:"Watashi no namae wa Raji desu."},
  {jp:"にほんごをべんきょうしています。",en:"I am studying Japanese.",romaji:"Nihongo wo benkyou shite imasu."},
  {jp:"すしはおいしいです。",en:"Sushi is delicious.",romaji:"Sushi wa oishii desu."},
  {jp:"きょうはいいてんきです。",en:"Today is good weather.",romaji:"Kyou wa ii tenki desu."},
  {jp:"どこにいきますか？",en:"Where are you going?",romaji:"Doko ni ikimasu ka?"},
  {jp:"トイレはどこですか？",en:"Where is the restroom?",romaji:"Toire wa doko desu ka?"},
  {jp:"これはいくらですか？",en:"How much is this?",romaji:"Kore wa ikura desu ka?"},
  {jp:"わかりません。",en:"I don't understand.",romaji:"Wakarimasen."},
  {jp:"もういちどいってください。",en:"Please say it again.",romaji:"Mou ichido itte kudasai."},
  {jp:"ゆっくりはなしてください。",en:"Please speak slowly.",romaji:"Yukkuri hanashite kudasai."},
];

const TONGUE_TWISTERS = [
  {text:"なまむぎ なまごめ なまたまご",romaji:"Namamugi namagome namatamago",en:"Raw wheat, raw rice, raw eggs"},
  {text:"となりのきゃくはよくかきくうきゃくだ",romaji:"Tonari no kyaku wa yoku kaki kuu kyaku da",en:"The guest next door is a guest who eats a lot of persimmons"},
  {text:"すもももももももものうち",romaji:"Sumomo mo momo mo momo no uchi",en:"Plums and peaches are both types of peaches"},
];

const CULTUREFACTS = [
  {emoji:"🎎",title:"Keigo",body:"Japanese has formal (keigo) and casual speech. Always use polite forms (〜ます/〜です) with strangers!"},
  {emoji:"🙇",title:"Bowing",body:"Bowing (おじぎ, ojigi) is essential. Deeper bow = more respect. No need to shake hands!"},
  {emoji:"🗾",title:"Hiragana First",body:"Japanese learners always start with hiragana. Master it before kanji — it's the foundation of everything!"},
  {emoji:"🔢",title:"Counters",body:"Japanese uses special counter words: 一枚 (ichimai) for flat things, 一本 (ippon) for long things, 一匹 (ippiki) for small animals!"},
  {emoji:"🍱",title:"Itadakimasu",body:"'いただきます' (itadakimasu) is said before eating — it means 'I humbly receive.' Always say it!"},
  {emoji:"🎌",title:"Furigana",body:"Small hiragana above kanji is called furigana. It helps beginners read kanji characters!"},
];

// ─── UTILITIES ───────────────────────────────────────────────────────────────

// Global sound settings (read by speak() and sfx()). Updated from the Sound Settings panel.
const soundSettings = { rate: 0.8, pitch: 1, volume: 1, voiceURI: null, sfxOn: true, sfxVolume: 0.35 };

function getJapaneseVoices() {
  if (!("speechSynthesis" in window)) return [];
  return window.speechSynthesis.getVoices().filter(v => v.lang && v.lang.toLowerCase().startsWith("ja"));
}

function speak(text, rateOverride) {
  if (!text) return;
  if ("speechSynthesis" in window) {
    window.speechSynthesis.cancel();
    const u = new SpeechSynthesisUtterance(text);
    u.lang = "ja-JP";
    u.rate = rateOverride != null ? rateOverride : soundSettings.rate;
    u.pitch = soundSettings.pitch;
    u.volume = soundSettings.volume;
    const voices = getJapaneseVoices();
    const chosen = soundSettings.voiceURI
      ? voices.find(v => v.voiceURI === soundSettings.voiceURI)
      : voices[0];
    if (chosen) u.voice = chosen;
    window.speechSynthesis.speak(u);
  }
}

// Lightweight WebAudio sound effects (no external files, never expires/rate-limited).
let _actx = null;
function getAudioCtx() {
  if (!_actx) {
    const AC = window.AudioContext || window.webkitAudioContext;
    if (AC) _actx = new AC();
  }
  return _actx;
}
function sfx(type) {
  if (!soundSettings.sfxOn) return;
  const ctx = getAudioCtx();
  if (!ctx) return;
  if (ctx.state === "suspended") ctx.resume();
  const now = ctx.currentTime;
  const vol = soundSettings.sfxVolume;

  const tone = (freq, start, dur, gainPeak, shape = "sine") => {
    const osc = ctx.createOscillator();
    const gain = ctx.createGain();
    osc.type = shape;
    osc.frequency.setValueAtTime(freq, now + start);
    gain.gain.setValueAtTime(0, now + start);
    gain.gain.linearRampToValueAtTime(gainPeak * vol, now + start + 0.015);
    gain.gain.exponentialRampToValueAtTime(0.0001, now + start + dur);
    osc.connect(gain); gain.connect(ctx.destination);
    osc.start(now + start); osc.stop(now + start + dur + 0.02);
  };

  switch (type) {
    case "click": tone(740, 0, 0.07, 0.18); break;
    case "flip": tone(520, 0, 0.09, 0.15); tone(680, 0.05, 0.08, 0.12); break;
    case "correct": tone(660, 0, 0.1, 0.22); tone(880, 0.08, 0.16, 0.22); break;
    case "wrong": tone(300, 0, 0.18, 0.2, "sawtooth"); break;
    case "streak": tone(523, 0, 0.08, 0.2); tone(659, 0.07, 0.08, 0.2); tone(784, 0.14, 0.14, 0.22); break;
    case "complete": tone(523, 0, 0.1, 0.2); tone(659, 0.1, 0.1, 0.2); tone(784, 0.2, 0.1, 0.2); tone(1047, 0.3, 0.2, 0.24); break;
    case "tick": tone(880, 0, 0.04, 0.1); break;
    case "mark": tone(990, 0, 0.06, 0.18); tone(1320, 0.05, 0.08, 0.16); break;
    case "swoosh": tone(380, 0, 0.1, 0.12, "triangle"); break;
    default: tone(600, 0, 0.07, 0.15);
  }
}

function useLocalStorage(key, init) {
  const [val, setVal] = useState(() => {
    try { const s = localStorage.getItem(key); return s ? JSON.parse(s) : init; }
    catch { return init; }
  });
  const set = useCallback(v => { setVal(v); try { localStorage.setItem(key, JSON.stringify(v)); } catch {} }, [key]);
  return [val, set];
}

// ─── SMALL COMPONENTS ────────────────────────────────────────────────────────

const Pill = ({label,active,color,onClick}) => (
  <button onClick={()=>{sfx("click");onClick();}} style={{
    padding:"6px 14px", borderRadius:20, border:`1.5px solid ${active?color:"#e0e0e0"}`,
    background: active ? color : "#fff", color: active?"#fff":"#555",
    fontSize:13, fontWeight:active?700:400, cursor:"pointer", whiteSpace:"nowrap", transition:"all 0.2s"
  }}>{label}</button>
);

const Badge = ({label,color}) => (
  <span style={{background:color+"22",color,fontSize:11,fontWeight:700,padding:"2px 8px",borderRadius:10,letterSpacing:.5}}>{label}</span>
);

const Card = ({children,style={}}) => (
  <div style={{background:"#fff",borderRadius:18,padding:18,boxShadow:"0 2px 12px rgba(0,0,0,0.06)",marginBottom:14,...style}}>{children}</div>
);

// ─── TOAST ───────────────────────────────────────────────────────────────────

function Toast({msg,color}) {
  return msg ? (
    <div style={{
      position:"fixed",top:16,left:"50%",transform:"translateX(-50%)",
      background:color||"#333",color:"#fff",padding:"10px 22px",
      borderRadius:20,fontSize:14,fontWeight:600,zIndex:9999,
      boxShadow:"0 4px 20px rgba(0,0,0,0.2)",pointerEvents:"none",
      animation:"fadeIn 0.2s ease"
    }}>{msg}</div>
  ) : null;
}

// ─── FLASHCARD ───────────────────────────────────────────────────────────────

function FlashCard({front,back,sub,color,onNext,onPrev,idx,total,onMark,marked}) {
  const [flipped, setFlipped] = useState(false);
  useEffect(()=>setFlipped(false),[front]);
  return (
    <div>
      <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:12}}>
        <span style={{fontSize:13,color:"#aaa"}}>{idx+1}/{total}</span>
        <button onClick={()=>{sfx("click");speak(front);}} style={{
          background:"#f5f5f5",border:"none",borderRadius:10,padding:"6px 12px",cursor:"pointer",fontSize:16
        }}>🔊 Listen</button>
        <button onClick={()=>{sfx("mark");onMark();}} style={{
          background: marked?"#fef3c7":"#f5f5f5",border:"none",borderRadius:10,
          padding:"6px 12px",cursor:"pointer",fontSize:16
        }}>{marked?"⭐":"☆"}</button>
      </div>
      <div onClick={()=>{sfx("flip");setFlipped(f=>!f);}} style={{perspective:1000,cursor:"pointer",marginBottom:18}}>
        <div style={{
          position:"relative",paddingTop:"55%",transition:"transform 0.5s cubic-bezier(.4,0,.2,1)",
          transformStyle:"preserve-3d",transform:flipped?"rotateY(180deg)":"rotateY(0)"
        }}>
          {[{side:"front",content:(
            <div style={{display:"flex",flexDirection:"column",alignItems:"center",justifyContent:"center",height:"100%",gap:8,padding:24}}>
              <div style={{fontSize:76,fontWeight:700,color,lineHeight:1}}>{front}</div>
              {sub && <div style={{fontSize:14,color:"#999"}}>{sub}</div>}
              <div style={{fontSize:12,color:"#ccc",marginTop:8}}>tap to flip</div>
            </div>
          )},{side:"back",content:(
            <div style={{display:"flex",flexDirection:"column",alignItems:"center",justifyContent:"center",height:"100%",gap:10,padding:24}}>
              <div style={{fontSize:38,fontWeight:700,color:"#fff",lineHeight:1}}>{front}</div>
              {back.map((b,i)=>(
                <div key={i} style={{textAlign:"center"}}>
                  <div style={{fontSize:i===0?24:15,fontWeight:i===0?700:400,color:"#fff",opacity:i===0?1:0.85}}>{b}</div>
                </div>
              ))}
            </div>
          )}].map(({side,content})=>(
            <div key={side} style={{
              position:"absolute",inset:0,backfaceVisibility:"hidden",
              transform:side==="back"?"rotateY(180deg)":"rotateY(0)",
              borderRadius:20,
              background:side==="back"?`linear-gradient(135deg,${color},${color}cc)`:`linear-gradient(135deg,${color}11,${color}22)`,
              border:`2px solid ${color}44`, display:"flex"
            }}>{content}</div>
          ))}
        </div>
      </div>
      <div style={{display:"flex",justifyContent:"center",gap:12}}>
        <button onClick={()=>{sfx("click");onPrev();}} disabled={idx===0} style={{
          flex:1,padding:"12px",borderRadius:14,border:"2px solid #e8e8e8",
          background:"#fff",fontSize:18,cursor:"pointer",opacity:idx===0?.4:1
        }}>←</button>
        <button onClick={()=>{sfx("swoosh");onNext();}} style={{
          flex:2,padding:"12px",borderRadius:14,border:"none",
          background:`linear-gradient(90deg,${color},#7C3AED)`,
          color:"#fff",fontSize:16,fontWeight:700,cursor:"pointer"
        }}>Next →</button>
      </div>
    </div>
  );
}

// ─── HIRAGANA / KATAKANA CHART ────────────────────────────────────────────────

function KanaChart({data,title,color,progress,onMark}) {
  const [selected, setSelected] = useState(null);
  const [filterLearned, setFilterLearned] = useState(false);
  const display = filterLearned ? data.filter((_,i)=>progress[i]) : data;
  return (
    <div>
      <div style={{display:"flex",gap:8,marginBottom:14,alignItems:"center",flexWrap:"wrap"}}>
        <Pill label="All" active={!filterLearned} color={color} onClick={()=>setFilterLearned(false)} />
        <Pill label="⭐ Marked" active={filterLearned} color={color} onClick={()=>setFilterLearned(true)} />
        <span style={{marginLeft:"auto",fontSize:12,color:"#999"}}>
          {Object.keys(progress).length}/{data.length} learned
        </span>
      </div>
      <div style={{display:"grid",gridTemplateColumns:"repeat(5,1fr)",gap:8,marginBottom:20}}>
        {display.map((k,i)=>{
          const realIdx = filterLearned ? data.indexOf(k) : i;
          const isLearned = progress[realIdx];
          const isSel = selected?.romaji === k.romaji;
          return (
            <div key={i} onClick={()=>{sfx("click");setSelected(isSel?null:k);}}
              style={{
                borderRadius:12,padding:"10px 4px",textAlign:"center",cursor:"pointer",
                background: isSel ? color : isLearned ? color+"22" : "#f8f8f8",
                border:`2px solid ${isSel?color:isLearned?color+"44":"transparent"}`,
                transition:"all 0.15s"
              }}>
              <div style={{fontSize:22,fontWeight:700,color:isSel?"#fff":"#222"}}>{k.char}</div>
              <div style={{fontSize:10,color:isSel?"#fff":"#888"}}>{k.romaji}</div>
            </div>
          );
        })}
      </div>
      {selected && (
        <Card style={{border:`2px solid ${color}44`,background:`linear-gradient(135deg,${color}08,${color}18)`}}>
          <div style={{display:"flex",alignItems:"center",gap:16}}>
            <div style={{fontSize:64,fontWeight:700,color}}>{selected.char}</div>
            <div style={{flex:1}}>
              <div style={{fontSize:24,fontWeight:700,marginBottom:4}}>{selected.romaji}</div>
              <div style={{display:"flex",gap:8,flexWrap:"wrap"}}>
                <button onClick={()=>{sfx("click");speak(selected.char);}} style={{
                  padding:"6px 14px",borderRadius:10,background:color,color:"#fff",
                  border:"none",cursor:"pointer",fontSize:13,fontWeight:600
                }}>🔊 Play</button>
                <button onClick={()=>{sfx("mark");onMark(selected);}} style={{
                  padding:"6px 14px",borderRadius:10,
                  background:progress[data.indexOf(selected)]?"#fef3c7":"#f5f5f5",
                  border:"none",cursor:"pointer",fontSize:13
                }}>{progress[data.indexOf(selected)]?"⭐ Marked":"☆ Mark"}</button>
              </div>
            </div>
          </div>
        </Card>
      )}
    </div>
  );
}

// ─── QUIZ ENGINE ─────────────────────────────────────────────────────────────

function QuizEngine({pool, onDone, mode="mcq"}) {
  const shuffled = useRef([...pool].sort(()=>Math.random()-.5).slice(0,Math.min(10,pool.length)));
  const [idx, setIdx] = useState(0);
  const [score, setScore] = useState(0);
  const [selected, setSelected] = useState(null);
  const [typed, setTyped] = useState("");
  const [opts, setOpts] = useState([]);
  const [wrong, setWrong] = useState([]);
  const [streak, setStreak] = useState(0);
  const [maxStreak, setMaxStreak] = useState(0);
  const inputRef = useRef(null);

  const q = shuffled.current[idx];
  const total = shuffled.current.length;

  useEffect(()=>{
    if(!q) return;
    const correct = q;
    const others = pool.filter(x=>x.romaji!==correct.romaji).sort(()=>Math.random()-.5).slice(0,3);
    setOpts([...others,correct].sort(()=>Math.random()-.5));
    setSelected(null); setTyped("");
    if(mode==="type") setTimeout(()=>inputRef.current?.focus(),100);
  },[idx, q]);

  const submit = (ans) => {
    if(selected!==null) return;
    const isRight = ans.trim().toLowerCase() === q.romaji.toLowerCase();
    setSelected(ans);
    if(isRight){
      setScore(s=>s+1);
      setStreak(s=>{
        const ns=s+1;
        setMaxStreak(m=>Math.max(m,ns));
        if(ns>=3) sfx("streak"); else sfx("correct");
        return ns;
      });
    } else {
      sfx("wrong");
      setWrong(w=>[...w,q]);
      setStreak(0);
    }
    setTimeout(()=>{
      if(idx+1>=total){ sfx("complete"); onDone({score:isRight?score+1:score, total, wrong:isRight?wrong:[...wrong,q], maxStreak}); }
      else setIdx(i=>i+1);
    }, mode==="type"?600:900);
  };

  if(!q) return null;
  const isRight = selected!==null && selected.trim().toLowerCase()===q.romaji.toLowerCase();

  return (
    <div>
      <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:16}}>
        <div>
          <div style={{fontSize:13,color:"#999"}}>Question {idx+1}/{total}</div>
          {streak>1 && <div style={{fontSize:12,color:"#F59E0B",fontWeight:700}}>🔥 {streak} streak!</div>}
        </div>
        <div style={{fontSize:15,fontWeight:700,color:"#E84393"}}>Score: {score}</div>
      </div>
      <div style={{width:"100%",height:6,background:"#f0f0f0",borderRadius:3,marginBottom:20}}>
        <div style={{height:"100%",width:`${((idx)/total)*100}%`,background:"linear-gradient(90deg,#E84393,#7C3AED)",borderRadius:3,transition:"width 0.3s"}}/>
      </div>
      <div style={{textAlign:"center",marginBottom:24}}>
        <div onClick={()=>{sfx("click");speak(q.char||q.jp||q.kanji);}} style={{
          fontSize:72,cursor:"pointer",lineHeight:1.1,marginBottom:6,
          filter: selected!==null && !isRight ? "none" : "none"
        }}>{q.char||q.jp||q.kanji}</div>
        <div style={{fontSize:13,color:"#aaa"}}>
          {mode==="mcq" ? "What is the romaji?" : "Type the romaji:"}
        </div>
      </div>
      {mode==="mcq" ? (
        <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10}}>
          {opts.map((o,i)=>{
            const ic = o.romaji===q.romaji;
            const is = selected===o.romaji;
            let bg="#f8f8f8",col="#333",bdr="2px solid #e8e8e8";
            if(selected!==null){
              if(ic){bg="#d1fae5";col="#065f46";bdr="2px solid #059669";}
              else if(is){bg="#fee2e2";col="#991b1b";bdr="2px solid #dc2626";}
            }
            return (
              <button key={i} onClick={()=>submit(o.romaji)} style={{
                padding:"14px 8px",borderRadius:14,background:bg,color:col,
                border:bdr,fontSize:15,fontWeight:600,cursor:"pointer",transition:"all 0.15s"
              }}>{o.romaji}</button>
            );
          })}
        </div>
      ) : (
        <div>
          <input ref={inputRef} value={typed} onChange={e=>setTyped(e.target.value)}
            onKeyDown={e=>e.key==="Enter"&&typed.trim()&&submit(typed)}
            placeholder="Type romaji here..."
            style={{
              width:"100%",padding:"14px 16px",borderRadius:14,fontSize:16,
              border: selected!==null ? `2px solid ${isRight?"#059669":"#dc2626"}` : "2px solid #e0e0e0",
              background: selected!==null ? (isRight?"#d1fae5":"#fee2e2") : "#fff",
              outline:"none",fontFamily:"inherit",boxSizing:"border-box"
            }}/>
          <button onClick={()=>{if(typed.trim()){sfx("click");submit(typed);}}} style={{
            width:"100%",marginTop:10,padding:"13px",borderRadius:14,
            background:"linear-gradient(90deg,#E84393,#7C3AED)",
            color:"#fff",border:"none",fontSize:16,fontWeight:700,cursor:"pointer"
          }}>Check ✓</button>
          {selected!==null && !isRight && (
            <div style={{marginTop:8,textAlign:"center",color:"#dc2626",fontSize:14}}>
              Correct: <strong>{q.romaji}</strong>
            </div>
          )}
        </div>
      )}
    </div>
  );
}

function QuizResult({result, onRetry, onHome}) {
  const pct = Math.round((result.score/result.total)*100);
  const emoji = pct>=90?"🏆":pct>=70?"🎉":pct>=50?"👍":"📚";
  return (
    <div style={{textAlign:"center",padding:24}}>
      <div style={{fontSize:64,marginBottom:12}}>{emoji}</div>
      <div style={{fontSize:42,fontWeight:800,background:"linear-gradient(90deg,#E84393,#7C3AED)",WebkitBackgroundClip:"text",WebkitTextFillColor:"transparent",marginBottom:4}}>
        {result.score}/{result.total}
      </div>
      <div style={{fontSize:16,color:"#666",marginBottom:6}}>
        {pct>=90?"Sugoi! Amazing!":pct>=70?"Yoku dekimashita! Well done!":pct>=50?"Gambatte! Keep going!":"Mou ichido! Try again!"}
      </div>
      {result.maxStreak>1 && (
        <div style={{fontSize:13,color:"#F59E0B",fontWeight:700,marginBottom:12}}>🔥 Best streak: {result.maxStreak}</div>
      )}
      {result.wrong.length>0 && (
        <div style={{textAlign:"left",marginBottom:16}}>
          <div style={{fontSize:13,fontWeight:700,color:"#dc2626",marginBottom:8}}>Review these:</div>
          {result.wrong.map((w,i)=>(
            <div key={i} style={{display:"flex",justifyContent:"space-between",padding:"8px 12px",
              background:"#fff5f5",borderRadius:10,marginBottom:6,fontSize:14}}>
              <span style={{fontWeight:700}}>{w.char||w.jp||w.kanji}</span>
              <span style={{color:"#888"}}>{w.romaji}</span>
            </div>
          ))}
        </div>
      )}
      <div style={{display:"flex",gap:10}}>
        <button onClick={()=>{sfx("click");onRetry();}} style={{
          flex:1,padding:"12px",borderRadius:14,background:"#f5f5f5",
          border:"none",fontSize:15,fontWeight:700,cursor:"pointer"
        }}>Retry</button>
        <button onClick={()=>{sfx("click");onHome();}} style={{
          flex:1,padding:"12px",borderRadius:14,
          background:"linear-gradient(90deg,#E84393,#7C3AED)",
          color:"#fff",border:"none",fontSize:15,fontWeight:700,cursor:"pointer"
        }}>Done ✓</button>
      </div>
    </div>
  );
}

// ─── LISTENING PRACTICE ───────────────────────────────────────────────────────

function ListeningGame({pool,onDone}) {
  const shuffled = useRef([...pool].sort(()=>Math.random()-.5).slice(0,8));
  const [idx, setIdx] = useState(0);
  const [score, setScore] = useState(0);
  const [selected, setSelected] = useState(null);
  const [played, setPlayed] = useState(false);
  const [opts, setOpts] = useState([]);

  const q = shuffled.current[idx];
  useEffect(()=>{
    if(!q) return;
    const others = pool.filter(x=>(x.romaji||x.en)!==(q.romaji||q.en)).sort(()=>Math.random()-.5).slice(0,3);
    setOpts([...others,q].sort(()=>Math.random()-.5));
    setSelected(null); setPlayed(false);
  },[idx,q]);

  if(!q) return null;
  const qText = q.char||q.jp||q.kanji;
  const answer = q.en||q.romaji;

  const choose = (o) => {
    if(selected) return;
    const a = o.en||o.romaji;
    setSelected(a);
    const isRight = a===answer;
    if(isRight){ setScore(s=>s+1); sfx("correct"); } else { sfx("wrong"); }
    setTimeout(()=>{
      if(idx+1>=shuffled.current.length){ sfx("complete"); onDone({score:isRight?score+1:score,total:shuffled.current.length}); }
      else setIdx(i=>i+1);
    },900);
  };

  return (
    <div>
      <div style={{display:"flex",justifyContent:"space-between",marginBottom:16}}>
        <span style={{fontSize:13,color:"#999"}}>{idx+1}/{shuffled.current.length}</span>
        <span style={{fontSize:14,fontWeight:700,color:"#E84393"}}>Score: {score}</span>
      </div>
      <div style={{textAlign:"center",padding:"32px 0"}}>
        <button onClick={()=>{sfx("click");speak(qText);setPlayed(true);}} style={{
          width:90,height:90,borderRadius:"50%",
          background:played?"linear-gradient(135deg,#E84393,#7C3AED)":"#f0f0f0",
          border:"none",fontSize:36,cursor:"pointer",boxShadow:"0 4px 20px rgba(232,67,147,0.3)",
          transition:"all 0.2s"
        }}>🔊</button>
        <div style={{fontSize:13,color:"#aaa",marginTop:12}}>
          {played?"What did you hear?":"Tap to listen"}
        </div>
      </div>
      {played && (
        <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10}}>
          {opts.map((o,i)=>{
            const a = o.en||o.romaji;
            const ic = a===answer;
            const is = selected===a;
            let bg="#f8f8f8",col="#333",bdr="2px solid #e8e8e8";
            if(selected){
              if(ic){bg="#d1fae5";col="#065f46";bdr="2px solid #059669";}
              else if(is){bg="#fee2e2";col="#991b1b";bdr="2px solid #dc2626";}
            }
            return (
              <button key={i} onClick={()=>choose(o)} style={{
                padding:"14px 8px",borderRadius:14,background:bg,color:col,
                border:bdr,fontSize:13,fontWeight:600,cursor:"pointer"
              }}>{a}</button>
            );
          })}
        </div>
      )}
    </div>
  );
}

// ─── SPEED ROUND ─────────────────────────────────────────────────────────────

function SpeedRound({pool,onDone}) {
  const shuffled = useRef([...pool].sort(()=>Math.random()-.5).slice(0,15));
  const [idx, setIdx] = useState(0);
  const [score, setScore] = useState(0);
  const [timeLeft, setTimeLeft] = useState(3);
  const [gameOver, setGameOver] = useState(false);
  const [result, setResult] = useState(null);
  const [opts, setOpts] = useState([]);
  const [selected, setSelected] = useState(null);
  const timerRef = useRef(null);

  const q = shuffled.current[idx];

  const nextQ = useCallback((scored) => {
    clearInterval(timerRef.current);
    setSelected(null);
    const ni = idx+1;
    if(ni >= shuffled.current.length){ setGameOver(true); setResult({score:scored,total:shuffled.current.length}); return; }
    setIdx(ni);
    setTimeLeft(3);
  },[idx]);

  useEffect(()=>{
    if(gameOver||!q) return;
    const others = pool.filter(x=>x.romaji!==q.romaji).sort(()=>Math.random()-.5).slice(0,3);
    setOpts([...others,q].sort(()=>Math.random()-.5));
    setSelected(null);
  },[idx,q,gameOver]);

  useEffect(()=>{
    if(gameOver) return;
    timerRef.current = setInterval(()=>{
      setTimeLeft(t=>{
        if(t<=1){ sfx("wrong"); nextQ(score); return 3; }
        if(t<=2) sfx("tick");
        return t-1;
      });
    },1000);
    return ()=>clearInterval(timerRef.current);
  },[idx,gameOver]);

  const choose = (o) => {
    if(selected) return;
    clearInterval(timerRef.current);
    const correct = o.romaji===q.romaji;
    setSelected(o.romaji);
    const ns = correct?score+1:score;
    if(correct){ setScore(ns); sfx("correct"); } else { sfx("wrong"); }
    setTimeout(()=>nextQ(ns),600);
  };

  useEffect(()=>{
    if(gameOver) sfx("complete");
  },[gameOver]);

  if(gameOver && result) return (
    <div style={{textAlign:"center",padding:24}}>
      <div style={{fontSize:52,marginBottom:8}}>⚡</div>
      <div style={{fontSize:36,fontWeight:800,color:"#E84393",marginBottom:4}}>{result.score}/{result.total}</div>
      <div style={{color:"#666",marginBottom:20}}>Speed Round complete!</div>
      <button onClick={onDone} style={{
        padding:"12px 32px",borderRadius:14,
        background:"linear-gradient(90deg,#E84393,#7C3AED)",
        color:"#fff",border:"none",fontSize:16,fontWeight:700,cursor:"pointer"
      }}>Finish</button>
    </div>
  );
  if(!q) return null;

  const timerColor = timeLeft>1?"#10B981":timeLeft===1?"#F59E0B":"#dc2626";
  return (
    <div>
      <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:10}}>
        <span style={{fontSize:13,color:"#999"}}>{idx+1}/{shuffled.current.length}</span>
        <div style={{
          width:44,height:44,borderRadius:"50%",background:timerColor,
          color:"#fff",display:"flex",alignItems:"center",justifyContent:"center",
          fontSize:20,fontWeight:800,transition:"background 0.3s"
        }}>{timeLeft}</div>
        <span style={{fontSize:14,fontWeight:700,color:"#E84393"}}>⚡{score}</span>
      </div>
      <div style={{textAlign:"center",padding:"20px 0 24px"}}>
        <div style={{fontSize:76,fontWeight:700}}>{q.char||q.jp}</div>
      </div>
      <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10}}>
        {opts.map((o,i)=>{
          const ic=o.romaji===q.romaji,is=selected===o.romaji;
          let bg="#f8f8f8",col="#333",bdr="2px solid #e8e8e8";
          if(selected){if(ic){bg="#d1fae5";col="#065f46";bdr="2px solid #059669";}else if(is){bg="#fee2e2";col="#991b1b";bdr="2px solid #dc2626";}}
          return <button key={i} onClick={()=>choose(o)} style={{padding:"14px",borderRadius:14,background:bg,color:col,border:bdr,fontSize:15,fontWeight:600,cursor:"pointer"}}>{o.romaji}</button>;
        })}
      </div>
    </div>
  );
}

// ─── AI TUTOR ─────────────────────────────────────────────────────────────────

function AITutor() {
  const [msgs, setMsgs] = useState([
    {role:"assistant",text:"こんにちは！ I'm Yuki, your AI Japanese tutor! 🎌\n\nAsk me anything:\n• Word meanings & usage\n• Grammar explanations\n• Pronunciation tips\n• Cultural questions\n• Translation help\n\nNani wo benkyou shitai desu ka? (What do you want to study?)"}
  ]);
  const [input, setInput] = useState("");
  const [loading, setLoading] = useState(false);
  const [suggestions] = useState(["How do I say 'I love you'?","Explain は vs が","What is keigo?","How to count in Japanese?","Translate: good luck"]);
  const bottomRef = useRef(null);

  useEffect(()=>{bottomRef.current?.scrollIntoView({behavior:"smooth"});},[msgs]);

  const send = async (text) => {
    const msg = text||input.trim();
    if(!msg||loading) return;
    setInput(""); setLoading(true);
    setMsgs(m=>[...m,{role:"user",text:msg}]);
    try {
      const res = await fetch("https://api.anthropic.com/v1/messages",{
        method:"POST",
        headers:{"Content-Type":"application/json"},
        body:JSON.stringify({
          model:"claude-sonnet-4-6", max_tokens:1000,
          system:`You are Yuki, a warm, expert Japanese language tutor for beginners to intermediate learners.
Rules:
- Always show Japanese in: かな/漢字 → romaji → English meaning format
- Keep explanations clear and concise
- Use encouraging language
- Give real example sentences
- When explaining grammar, show the pattern then examples
- Add fun cultural notes when relevant
- Use emojis occasionally to stay engaging
- If asked to translate, show the Japanese text prominently
- Correct misconceptions gently`,
          messages:[
            ...msgs.filter((_,i)=>i>0).map(m=>({role:m.role==="assistant"?"assistant":"user",content:m.text})),
            {role:"user",content:msg}
          ]
        })
      });
      const data = await res.json();
      setMsgs(m=>[...m,{role:"assistant",text:data.content?.[0]?.text||"Sumimasen, something went wrong!"}]);
    } catch {
      setMsgs(m=>[...m,{role:"assistant",text:"Network error. Please try again! 🙏"}]);
    }
    setLoading(false);
  };

  return (
    <div style={{display:"flex",flexDirection:"column",height:"100%"}}>
      <div style={{flex:1,overflowY:"auto",paddingRight:4,maxHeight:420}}>
        {msgs.map((m,i)=>(
          <div key={i} style={{display:"flex",justifyContent:m.role==="user"?"flex-end":"flex-start",marginBottom:10}}>
            {m.role==="assistant" && <div style={{width:28,height:28,borderRadius:"50%",background:"linear-gradient(135deg,#E84393,#7C3AED)",display:"flex",alignItems:"center",justifyContent:"center",fontSize:14,marginRight:8,flexShrink:0,marginTop:2}}>🎌</div>}
            <div style={{
              maxWidth:"80%",padding:"10px 14px",
              borderRadius:m.role==="user"?"18px 18px 4px 18px":"18px 18px 18px 4px",
              background:m.role==="user"?"linear-gradient(135deg,#E84393,#7C3AED)":"#f0f0f0",
              color:m.role==="user"?"#fff":"#222",
              fontSize:14,lineHeight:1.6,whiteSpace:"pre-wrap",wordBreak:"break-word"
            }}>{m.text}</div>
          </div>
        ))}
        {loading && (
          <div style={{display:"flex",alignItems:"center",gap:8,padding:"8px 0"}}>
            <div style={{width:28,height:28,borderRadius:"50%",background:"linear-gradient(135deg,#E84393,#7C3AED)",display:"flex",alignItems:"center",justifyContent:"center",fontSize:14}}>🎌</div>
            <div style={{display:"flex",gap:4,padding:"10px 14px",background:"#f0f0f0",borderRadius:"18px 18px 18px 4px"}}>
              {[0,1,2].map(i=><div key={i} style={{width:7,height:7,borderRadius:"50%",background:"#aaa",animation:"bounce 1s infinite",animationDelay:`${i*0.2}s`}}/>)}
            </div>
          </div>
        )}
        <div ref={bottomRef}/>
      </div>
      <div style={{overflowX:"auto",display:"flex",gap:6,paddingBottom:10,marginBottom:8}}>
        {suggestions.map((s,i)=>(
          <button key={i} onClick={()=>{sfx("click");send(s);}} style={{
            whiteSpace:"nowrap",padding:"6px 12px",borderRadius:16,
            background:"#f0f0f0",border:"none",fontSize:12,cursor:"pointer",color:"#555"
          }}>{s}</button>
        ))}
      </div>
      <div style={{display:"flex",gap:8}}>
        <input value={input} onChange={e=>setInput(e.target.value)}
          onKeyDown={e=>e.key==="Enter"&&send()}
          placeholder="Ask Yuki anything..."
          style={{flex:1,padding:"12px 16px",borderRadius:14,border:"2px solid #e0e0e0",fontSize:14,outline:"none",fontFamily:"inherit"}}/>
        <button onClick={()=>{sfx("swoosh");send();}} disabled={loading||!input.trim()} style={{
          padding:"12px 16px",borderRadius:14,
          background:"linear-gradient(135deg,#E84393,#7C3AED)",
          color:"#fff",border:"none",cursor:"pointer",fontSize:18,fontWeight:700,
          opacity:loading||!input.trim()?.5:1
        }}>→</button>
      </div>
    </div>
  );
}

// ─── SENTENCE BUILDER ─────────────────────────────────────────────────────────

function SentenceBuilder() {
  const [current, setCurrent] = useState(()=>SENTENCES[Math.floor(Math.random()*SENTENCES.length)]);
  const [words, setWords] = useState([]);
  const [pool, setPool] = useState([]);
  const [checked, setChecked] = useState(null);

  const reset = (s) => {
    const sentence = s||current;
    const parts = sentence.jp.split("").filter(c=>c.trim());
    // Make word-level chunks (simple split by known words)
    const chunks = sentence.romaji.split(" ").map((r,i)=>({r,jp:sentence.jp}));
    // Use romaji words as pieces
    const pieces = sentence.romaji.split(" ");
    setPool([...pieces].sort(()=>Math.random()-.5));
    setWords([]);
    setChecked(null);
  };

  useEffect(()=>reset(),[]);

  const addWord = (w,i) => {
    if(checked!==null) return;
    sfx("click");
    setWords(prev=>[...prev,w]);
    setPool(prev=>{const n=[...prev];n.splice(i,1);return n;});
  };
  const removeWord = (i) => {
    if(checked!==null) return;
    sfx("click");
    setPool(prev=>[...prev,words[i]]);
    setWords(prev=>{const n=[...prev];n.splice(i,1);return n;});
  };
  const check = () => {
    const correct = current.romaji.split(" ");
    const isRight = words.join(" ")===correct.join(" ");
    setChecked(isRight);
    sfx(isRight?"correct":"wrong");
  };
  const next = () => {
    sfx("swoosh");
    const s = SENTENCES[Math.floor(Math.random()*SENTENCES.length)];
    setCurrent(s); reset(s);
  };

  return (
    <div>
      <Card style={{border:"2px solid #E84393",background:"#fff5fa"}}>
        <div style={{fontSize:13,color:"#E84393",fontWeight:700,marginBottom:4}}>Translate to Japanese:</div>
        <div style={{fontSize:18,fontWeight:700,color:"#222",marginBottom:4}}>{current.en}</div>
        <div style={{fontSize:12,color:"#aaa"}}>Hint: {current.jp}</div>
      </Card>
      {/* Drop zone */}
      <div style={{minHeight:56,border:"2px dashed #e0e0e0",borderRadius:14,padding:"10px 12px",marginBottom:14,display:"flex",flexWrap:"wrap",gap:6,background: checked===true?"#d1fae5":checked===false?"#fee2e2":"#fafafa"}}>
        {words.length===0 && <span style={{color:"#ccc",fontSize:14,alignSelf:"center"}}>Tap words below to build the sentence</span>}
        {words.map((w,i)=>(
          <button key={i} onClick={()=>removeWord(i)} style={{
            padding:"6px 12px",borderRadius:10,background:"#fff",
            border:"2px solid #E84393",color:"#E84393",fontSize:14,fontWeight:600,cursor:"pointer"
          }}>{w}</button>
        ))}
      </div>
      {/* Word pool */}
      <div style={{display:"flex",flexWrap:"wrap",gap:8,marginBottom:16}}>
        {pool.map((w,i)=>(
          <button key={i} onClick={()=>addWord(w,i)} style={{
            padding:"8px 14px",borderRadius:12,background:"#f0f0f0",
            border:"2px solid #e0e0e0",color:"#333",fontSize:14,fontWeight:600,cursor:"pointer"
          }}>{w}</button>
        ))}
      </div>
      {checked!==null && (
        <div style={{textAlign:"center",marginBottom:12,fontSize:15,fontWeight:700,color:checked?"#059669":"#dc2626"}}>
          {checked?"✅ Correct! Yatta!":"❌ Not quite — see the hint above"}
        </div>
      )}
      <div style={{display:"flex",gap:8}}>
        {checked===null ? (
          <button onClick={check} disabled={words.length===0} style={{
            flex:1,padding:"13px",borderRadius:14,
            background:"linear-gradient(90deg,#E84393,#7C3AED)",
            color:"#fff",border:"none",fontSize:15,fontWeight:700,cursor:"pointer",
            opacity:words.length===0?.5:1
          }}>Check ✓</button>
        ) : (
          <button onClick={next} style={{
            flex:1,padding:"13px",borderRadius:14,
            background:"linear-gradient(90deg,#E84393,#7C3AED)",
            color:"#fff",border:"none",fontSize:15,fontWeight:700,cursor:"pointer"
          }}>Next Sentence →</button>
        )}
        <button onClick={()=>reset()} style={{
          padding:"13px",borderRadius:14,background:"#f5f5f5",border:"none",cursor:"pointer",fontSize:18
        }}>🔄</button>
      </div>
    </div>
  );
}

// ─── TONGUE TWISTER ───────────────────────────────────────────────────────────

function TongueTwisters() {
  const [idx, setIdx] = useState(0);
  const [playing, setPlaying] = useState(false);
  const t = TONGUE_TWISTERS[idx];
  const playSlowly = () => {
    setPlaying(true);
    speak(t.text, 0.5);
    setTimeout(()=>setPlaying(false), 4000);
  };
  return (
    <div>
      <div style={{fontSize:14,color:"#666",marginBottom:14}}>Master these to perfect your pronunciation! 🗣️</div>
      {TONGUE_TWISTERS.map((tw,i)=>(
        <Card key={i} style={{border:i===idx?"2px solid #E84393":"2px solid transparent",cursor:"pointer"}} >
          <div onClick={()=>{sfx("click");setIdx(i);}}>
            <div style={{fontSize:20,fontWeight:700,color:"#222",marginBottom:4}}>{tw.text}</div>
            <div style={{fontSize:13,color:"#888",marginBottom:2}}>{tw.romaji}</div>
            <div style={{fontSize:12,color:"#aaa",fontStyle:"italic"}}>{tw.en}</div>
          </div>
          {i===idx && (
            <div style={{display:"flex",gap:8,marginTop:12}}>
              <button onClick={()=>{sfx("click");speak(tw.text,0.8);}} style={{flex:1,padding:"10px",borderRadius:12,background:"#E84393",color:"#fff",border:"none",cursor:"pointer",fontWeight:600}}>🔊 Normal</button>
              <button onClick={()=>{sfx("click");playSlowly();}} style={{flex:1,padding:"10px",borderRadius:12,background:"#7C3AED",color:"#fff",border:"none",cursor:"pointer",fontWeight:600}}>🐢 Slow</button>
            </div>
          )}
        </Card>
      ))}
    </div>
  );
}

// ─── KANJI SECTION ────────────────────────────────────────────────────────────

function KanjiLearn({progress,onMark}) {
  const [selected, setSelected] = useState(null);
  return (
    <div>
      <div style={{fontSize:13,color:"#666",marginBottom:14}}>Basic Kanji for beginners (JLPT N5 level)</div>
      <div style={{display:"grid",gridTemplateColumns:"repeat(4,1fr)",gap:10,marginBottom:16}}>
        {KANJI.map((k,i)=>(
          <div key={i} onClick={()=>{sfx("click");setSelected(selected?.kanji===k.kanji?null:k);}}
            style={{
              borderRadius:14,padding:"12px 6px",textAlign:"center",cursor:"pointer",
              background: selected?.kanji===k.kanji?"linear-gradient(135deg,#E84393,#7C3AED)": progress[i]?"#fff0f8":"#f8f8f8",
              border:`2px solid ${selected?.kanji===k.kanji?"#E84393":progress[i]?"#E84393":"transparent"}`,
              transition:"all 0.15s"
            }}>
            <div style={{fontSize:28,fontWeight:700,color:selected?.kanji===k.kanji?"#fff":"#222"}}>{k.kanji}</div>
            <div style={{fontSize:10,color:selected?.kanji===k.kanji?"#fff":"#888"}}>{k.en}</div>
          </div>
        ))}
      </div>
      {selected && (
        <Card style={{border:"2px solid #E84393",background:"linear-gradient(135deg,#fff5fa,#f5f0ff)"}}>
          <div style={{display:"flex",alignItems:"center",gap:16,marginBottom:12}}>
            <div style={{fontSize:64,fontWeight:700,color:"#E84393"}}>{selected.kanji}</div>
            <div style={{flex:1}}>
              <div style={{fontSize:18,fontWeight:700}}>{selected.en}</div>
              <div style={{fontSize:13,color:"#888"}}>Strokes: {selected.stroke}</div>
            </div>
          </div>
          <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10,marginBottom:12}}>
            <div style={{background:"#fff",borderRadius:10,padding:10}}>
              <div style={{fontSize:11,color:"#aaa",fontWeight:700,marginBottom:2}}>ON reading</div>
              <div style={{fontSize:16,fontWeight:700,color:"#7C3AED"}}>{selected.on}</div>
            </div>
            <div style={{background:"#fff",borderRadius:10,padding:10}}>
              <div style={{fontSize:11,color:"#aaa",fontWeight:700,marginBottom:2}}>KUN reading</div>
              <div style={{fontSize:16,fontWeight:700,color:"#E84393"}}>{selected.kun}</div>
            </div>
          </div>
          <div style={{background:"#fff",borderRadius:10,padding:10,marginBottom:12}}>
            <div style={{fontSize:11,color:"#aaa",fontWeight:700,marginBottom:4}}>Example</div>
            <div style={{fontSize:15,fontWeight:600}}>{selected.ex}</div>
          </div>
          <div style={{display:"flex",gap:8}}>
            <button onClick={()=>{sfx("click");speak(selected.kanji);}} style={{flex:1,padding:"10px",borderRadius:12,background:"#E84393",color:"#fff",border:"none",cursor:"pointer",fontWeight:600}}>🔊 Listen</button>
            <button onClick={()=>{sfx("mark");onMark(KANJI.indexOf(selected));}} style={{
              flex:1,padding:"10px",borderRadius:12,
              background:progress[KANJI.indexOf(selected)]?"#fef3c7":"#f5f5f5",
              border:"none",cursor:"pointer",fontWeight:600
            }}>{progress[KANJI.indexOf(selected)]?"⭐ Marked":"☆ Mark"}</button>
          </div>
        </Card>
      )}
    </div>
  );
}

// ─── PROGRESS DASHBOARD ───────────────────────────────────────────────────────

function Dashboard({progress,streakDays,totalQuizzes,totalCorrect}) {
  const hPct = Math.round((Object.keys(progress.hiragana||{}).length/HIRAGANA.length)*100);
  const kPct = Math.round((Object.keys(progress.katakana||{}).length/KATAKANA.length)*100);
  const kanjiPct = Math.round((Object.keys(progress.kanji||{}).length/KANJI.length)*100);
  const stats = [
    {label:"Hiragana",pct:hPct,color:"#E84393",icon:"ひ"},
    {label:"Katakana",pct:kPct,color:"#7C3AED",icon:"カ"},
    {label:"Kanji",pct:kanjiPct,color:"#F59E0B",icon:"漢"},
  ];
  const accuracy = totalQuizzes>0?Math.round((totalCorrect/totalQuizzes)*100):0;
  return (
    <div>
      <div style={{display:"grid",gridTemplateColumns:"1fr 1fr 1fr",gap:10,marginBottom:16}}>
        {[
          {label:"Day Streak",val:`${streakDays}🔥`,color:"#F59E0B"},
          {label:"Quizzes",val:totalQuizzes,color:"#10B981"},
          {label:"Accuracy",val:`${accuracy}%`,color:"#E84393"},
        ].map((s,i)=>(
          <Card key={i} style={{textAlign:"center",padding:14}}>
            <div style={{fontSize:22,fontWeight:800,color:s.color}}>{s.val}</div>
            <div style={{fontSize:11,color:"#888",marginTop:2}}>{s.label}</div>
          </Card>
        ))}
      </div>
      <div style={{fontSize:14,fontWeight:700,marginBottom:10}}>Learning Progress</div>
      {stats.map((s,i)=>(
        <div key={i} style={{marginBottom:14}}>
          <div style={{display:"flex",justifyContent:"space-between",marginBottom:4}}>
            <div style={{display:"flex",alignItems:"center",gap:6}}>
              <span style={{fontSize:16}}>{s.icon}</span>
              <span style={{fontSize:14,fontWeight:600}}>{s.label}</span>
            </div>
            <span style={{fontSize:13,color:s.color,fontWeight:700}}>{s.pct}%</span>
          </div>
          <div style={{height:10,background:"#f0f0f0",borderRadius:6,overflow:"hidden"}}>
            <div style={{height:"100%",width:`${s.pct}%`,background:`linear-gradient(90deg,${s.color},${s.color}aa)`,borderRadius:6,transition:"width 0.5s"}}/>
          </div>
        </div>
      ))}
      <div style={{fontSize:14,fontWeight:700,marginBottom:10,marginTop:4}}>Culture Notes</div>
      {CULTUREFACTS.map((f,i)=>(
        <Card key={i}>
          <div style={{display:"flex",gap:12,alignItems:"flex-start"}}>
            <span style={{fontSize:28}}>{f.emoji}</span>
            <div>
              <div style={{fontSize:14,fontWeight:700,marginBottom:2}}>{f.title}</div>
              <div style={{fontSize:13,color:"#555",lineHeight:1.5}}>{f.body}</div>
            </div>
          </div>
        </Card>
      ))}
    </div>
  );
}

// ─── SOUND SETTINGS PANEL ─────────────────────────────────────────────────────

function SoundSettingsPanel() {
  const [rate, setRate] = useState(soundSettings.rate);
  const [pitch, setPitch] = useState(soundSettings.pitch);
  const [volume, setVolume] = useState(soundSettings.volume);
  const [sfxOn, setSfxOn] = useState(soundSettings.sfxOn);
  const [sfxVolume, setSfxVolume] = useState(soundSettings.sfxVolume);
  const [voices, setVoices] = useState([]);
  const [voiceURI, setVoiceURI] = useState(soundSettings.voiceURI);

  useEffect(() => {
    const loadVoices = () => setVoices(getJapaneseVoices());
    loadVoices();
    if ("speechSynthesis" in window) {
      window.speechSynthesis.onvoiceschanged = loadVoices;
    }
  }, []);

  useEffect(() => { soundSettings.rate = rate; }, [rate]);
  useEffect(() => { soundSettings.pitch = pitch; }, [pitch]);
  useEffect(() => { soundSettings.volume = volume; }, [volume]);
  useEffect(() => { soundSettings.sfxOn = sfxOn; }, [sfxOn]);
  useEffect(() => { soundSettings.sfxVolume = sfxVolume; }, [sfxVolume]);
  useEffect(() => { soundSettings.voiceURI = voiceURI; }, [voiceURI]);

  const Row = ({label, value, suffix, children}) => (
    <div style={{marginBottom:16}}>
      <div style={{display:"flex",justifyContent:"space-between",marginBottom:6}}>
        <span style={{fontSize:13,fontWeight:600,color:"#444"}}>{label}</span>
        <span style={{fontSize:12,color:"#E84393",fontWeight:700}}>{value}{suffix}</span>
      </div>
      {children}
    </div>
  );

  const sliderStyle = {
    width:"100%", height:6, borderRadius:3, accentColor:"#E84393", cursor:"pointer"
  };

  return (
    <Card>
      {/* Voice selector */}
      <Row label="Japanese Voice" value={voices.length ? "" : "Using device default"}>
        {voices.length > 0 ? (
          <select value={voiceURI||""} onChange={e=>setVoiceURI(e.target.value||null)} style={{
            width:"100%",padding:"10px 12px",borderRadius:10,border:"2px solid #e8e8e8",
            fontSize:13,background:"#fff",color:"#333"
          }}>
            <option value="">Default ({voices[0]?.name})</option>
            {voices.map(v=>(
              <option key={v.voiceURI} value={v.voiceURI}>{v.name}</option>
            ))}
          </select>
        ) : (
          <div style={{fontSize:12,color:"#aaa",fontStyle:"italic"}}>No Japanese voices found on this device — using system default for ja-JP.</div>
        )}
      </Row>

      {/* Speech rate */}
      <Row label="🗣️ Speech Speed" value={rate.toFixed(2)} suffix="×">
        <input type="range" min="0.3" max="1.3" step="0.05" value={rate}
          onChange={e=>setRate(parseFloat(e.target.value))} style={sliderStyle}/>
      </Row>

      {/* Pitch */}
      <Row label="🎵 Voice Pitch" value={pitch.toFixed(2)} suffix="×">
        <input type="range" min="0.5" max="1.8" step="0.05" value={pitch}
          onChange={e=>setPitch(parseFloat(e.target.value))} style={sliderStyle}/>
      </Row>

      {/* TTS volume */}
      <Row label="🔊 Voice Volume" value={Math.round(volume*100)} suffix="%">
        <input type="range" min="0" max="1" step="0.05" value={volume}
          onChange={e=>setVolume(parseFloat(e.target.value))} style={sliderStyle}/>
      </Row>

      <div style={{height:1,background:"#f0f0f0",margin:"14px 0"}}/>

      {/* SFX toggle */}
      <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:sfxOn?14:0}}>
        <div>
          <div style={{fontSize:13,fontWeight:600,color:"#444"}}>🎮 UI Sound Effects</div>
          <div style={{fontSize:11,color:"#999"}}>Clicks, correct/wrong chimes, streaks</div>
        </div>
        <button onClick={()=>{const v=!sfxOn; setSfxOn(v); if(v) setTimeout(()=>sfx("correct"),50);}} style={{
          width:50,height:28,borderRadius:14,border:"none",cursor:"pointer",position:"relative",
          background: sfxOn ? "linear-gradient(90deg,#E84393,#7C3AED)" : "#e0e0e0", transition:"background 0.2s"
        }}>
          <div style={{
            position:"absolute",top:3,left: sfxOn?25:3,width:22,height:22,borderRadius:"50%",
            background:"#fff",transition:"left 0.2s",boxShadow:"0 1px 3px rgba(0,0,0,0.3)"
          }}/>
        </button>
      </div>

      {sfxOn && (
        <Row label="SFX Volume" value={Math.round(sfxVolume*100)} suffix="%">
          <input type="range" min="0" max="1" step="0.05" value={sfxVolume}
            onChange={e=>setSfxVolume(parseFloat(e.target.value))} style={sliderStyle}/>
        </Row>
      )}

      <div style={{height:1,background:"#f0f0f0",margin:"4px 0 14px"}}/>

      {/* Test buttons */}
      <div style={{display:"flex",gap:8}}>
        <button onClick={()=>speak("こんにちは、げんきですか？")} style={{
          flex:1,padding:"11px",borderRadius:12,background:"#E84393",color:"#fff",
          border:"none",cursor:"pointer",fontSize:13,fontWeight:600
        }}>🔊 Test Voice</button>
        <button onClick={()=>{sfx("correct");setTimeout(()=>sfx("streak"),500);}} style={{
          flex:1,padding:"11px",borderRadius:12,background:"#7C3AED",color:"#fff",
          border:"none",cursor:"pointer",fontSize:13,fontWeight:600
        }}>🎮 Test SFX</button>
      </div>
    </Card>
  );
}

// ─── MAIN APP ─────────────────────────────────────────────────────────────────

export default function App() {
  const [tab, setTab] = useState("home");
  const [subTab, setSubTab] = useState(null);
  const [toast, setToast] = useState(null);

  // Persistent progress
  const [hProgress, setHProgress] = useLocalStorage("jp_hira", {});
  const [kProgress, setKProgress] = useLocalStorage("jp_kata", {});
  const [kanjiProgress, setKanjiProgress] = useLocalStorage("jp_kanji", {});
  const [vocabProgress, setVocabProgress] = useLocalStorage("jp_vocab", {});
  const [streakDays] = useLocalStorage("jp_streak", 1);
  const [totalQuizzes, setTotalQuizzes] = useLocalStorage("jp_quizzes", 0);
  const [totalCorrect, setTotalCorrect] = useLocalStorage("jp_correct", 0);

  // Flashcard state
  const [fcSection, setFcSection] = useState("greetings");
  const [fcIdx, setFcIdx] = useState(0);

  // Quiz state
  const [quizMode, setQuizMode] = useState("mcq");
  const [quizSection, setQuizSection] = useState("hiragana");
  const [quizResult, setQuizResult] = useState(null);
  const [quizKey, setQuizKey] = useState(0);
  const [gameMode, setGameMode] = useState(null); // "quiz"|"listening"|"speed"

  // Kana chart state
  const [kanaType, setKanaType] = useState("hiragana");

  const showToast = (msg, color="#333") => {
    setToast({msg,color});
    setTimeout(()=>setToast(null), 2000);
  };

  const markHira = (k) => {
    const i = HIRAGANA.indexOf(k);
    setHProgress(p=>({...p,[i]:!p[i]}));
    showToast(hProgress[i]?"Unmarked":"⭐ Marked!","#E84393");
  };
  const markKata = (k) => {
    const i = KATAKANA.indexOf(k);
    setKProgress(p=>({...p,[i]:!p[i]}));
    showToast(kProgress[i]?"Unmarked":"⭐ Marked!","#7C3AED");
  };
  const markKanji = (i) => {
    setKanjiProgress(p=>({...p,[i]:!p[i]}));
    showToast(kanjiProgress[i]?"Unmarked":"⭐ Marked!","#F59E0B");
  };

  const vocabWords = VOCAB[fcSection]?.words || [];
  const fcItem = vocabWords[fcIdx];

  const getQuizPool = () => {
    if(quizSection==="hiragana") return HIRAGANA;
    if(quizSection==="katakana") return KATAKANA;
    if(quizSection==="vocab") return Object.values(VOCAB).flatMap(v=>v.words).map(w=>({char:w.jp,romaji:w.romaji,en:w.en}));
    return [...HIRAGANA,...KATAKANA];
  };

  const TABS = [
    {id:"home",icon:"🏠",label:"Home"},
    {id:"kana",icon:"あ",label:"Kana"},
    {id:"vocab",icon:"📖",label:"Vocab"},
    {id:"kanji",icon:"漢",label:"Kanji"},
    {id:"quiz",icon:"🎯",label:"Quiz"},
    {id:"ai",icon:"🤖",label:"Tutor"},
    {id:"more",icon:"⚙️",label:"More"},
  ];

  return (
    <div style={{fontFamily:"'Segoe UI',system-ui,sans-serif",background:"#f5f5f5",minHeight:"100vh",maxWidth:480,margin:"0 auto",position:"relative"}}>
      <style>{`
        @keyframes bounce{0%,100%{transform:translateY(0)}50%{transform:translateY(-5px)}}
        @keyframes fadeIn{from{opacity:0;transform:translateY(8px)}to{opacity:1;transform:translateY(0)}}
        *{box-sizing:border-box}
        ::-webkit-scrollbar{width:3px}
        ::-webkit-scrollbar-thumb{background:#ddd;border-radius:3px}
        input,button{font-family:inherit}
      `}</style>

      {toast && <Toast msg={toast.msg} color={toast.color}/>}

      {/* HEADER */}
      <div style={{background:"linear-gradient(135deg,#E84393 0%,#7C3AED 100%)",padding:"18px 18px 22px",color:"#fff",position:"sticky",top:0,zIndex:100}}>
        <div style={{display:"flex",justifyContent:"space-between",alignItems:"center"}}>
          <div>
            <div style={{fontSize:11,opacity:.7,letterSpacing:2,textTransform:"uppercase"}}>日本語を学ぼう</div>
            <div style={{fontSize:20,fontWeight:800}}>
              {tab==="home"?"Learn Japanese":
               tab==="kana"?"Kana Charts":
               tab==="vocab"?"Vocabulary":
               tab==="kanji"?"Kanji":
               tab==="quiz"?"Practice Games":
               tab==="ai"?"AI Tutor · Yuki":
               "More Features"}
            </div>
          </div>
          <div style={{fontSize:28}}>🎌</div>
        </div>
      </div>

      {/* CONTENT */}
      <div style={{padding:"16px 14px",paddingBottom:90,animation:"fadeIn 0.25s ease"}}>

        {/* ── HOME ── */}
        {tab==="home" && (
          <div>
            {/* Welcome card */}
            <div style={{background:"linear-gradient(135deg,#fff5fa,#f5f0ff)",borderRadius:20,padding:18,marginBottom:16,border:"2px solid #E84393"}}>
              <div style={{fontSize:16,fontWeight:700,marginBottom:4}}>こんにちは、Raj! 👋</div>
              <div style={{fontSize:13,color:"#555",lineHeight:1.6}}>Your Japanese learning journey awaits. Start with Hiragana and build up from there!</div>
              <div style={{marginTop:10,display:"flex",gap:8,flexWrap:"wrap"}}>
                <Badge label="Beginner Friendly" color="#E84393"/>
                <Badge label="46 Kana + Kanji" color="#7C3AED"/>
                <Badge label="AI Tutor" color="#10B981"/>
              </div>
            </div>
            {/* Daily tip */}
            <Card style={{background:"#fffbeb",border:"2px solid #fcd34d"}}>
              <div style={{fontSize:12,color:"#D97706",fontWeight:700,letterSpacing:1,marginBottom:4}}>💡 TODAY'S TIP</div>
              <div style={{fontSize:14,color:"#444",lineHeight:1.6}}>Focus on hiragana first — you can read 46 syllables in 2 weeks with daily practice. Even 10 minutes a day works!</div>
            </Card>
            {/* Quick access */}
            <div style={{fontSize:14,fontWeight:700,marginBottom:10,marginTop:4}}>Quick Start</div>
            <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10,marginBottom:16}}>
              {[
                {label:"Hiragana Chart",sub:"46 characters",icon:"ひ",color:"#E84393",action:()=>{setTab("kana");setKanaType("hiragana");}},
                {label:"Katakana Chart",sub:"46 characters",icon:"カ",color:"#7C3AED",action:()=>{setTab("kana");setKanaType("katakana");}},
                {label:"Basic Vocab",sub:"Greetings, Food…",icon:"📖",color:"#10B981",action:()=>setTab("vocab")},
                {label:"Kanji Basics",sub:"15 N5 Kanji",icon:"漢",color:"#F59E0B",action:()=>setTab("kanji")},
                {label:"Quiz Me!",sub:"Test yourself",icon:"🎯",color:"#EC4899",action:()=>setTab("quiz")},
                {label:"Ask AI Tutor",sub:"Chat with Yuki",icon:"🤖",color:"#6366F1",action:()=>setTab("ai")},
              ].map((item,i)=>(
                <div key={i} onClick={()=>{sfx("click");item.action();}} style={{
                  background:"#fff",borderRadius:16,padding:14,cursor:"pointer",
                  border:`2px solid ${item.color}22`,boxShadow:"0 2px 8px rgba(0,0,0,0.05)",
                  display:"flex",gap:12,alignItems:"center"
                }}>
                  <div style={{width:44,height:44,borderRadius:12,background:item.color+"22",display:"flex",alignItems:"center",justifyContent:"center",fontSize:22,flexShrink:0}}>{item.icon}</div>
                  <div>
                    <div style={{fontSize:13,fontWeight:700,color:"#222"}}>{item.label}</div>
                    <div style={{fontSize:11,color:"#888"}}>{item.sub}</div>
                  </div>
                </div>
              ))}
            </div>
            <div style={{fontSize:14,fontWeight:700,marginBottom:10}}>Your Progress</div>
            <Dashboard
              progress={{hiragana:hProgress,katakana:kProgress,kanji:kanjiProgress}}
              streakDays={streakDays} totalQuizzes={totalQuizzes} totalCorrect={totalCorrect}
            />
          </div>
        )}

        {/* ── KANA ── */}
        {tab==="kana" && (
          <div>
            <div style={{display:"flex",gap:8,marginBottom:16}}>
              <Pill label="Hiragana ひ" active={kanaType==="hiragana"} color="#E84393" onClick={()=>setKanaType("hiragana")}/>
              <Pill label="Katakana カ" active={kanaType==="katakana"} color="#7C3AED" onClick={()=>setKanaType("katakana")}/>
            </div>
            {kanaType==="hiragana"
              ? <KanaChart data={HIRAGANA} title="Hiragana" color="#E84393" progress={hProgress} onMark={markHira}/>
              : <KanaChart data={KATAKANA} title="Katakana" color="#7C3AED" progress={kProgress} onMark={markKata}/>
            }
          </div>
        )}

        {/* ── VOCAB (Flashcards) ── */}
        {tab==="vocab" && (
          <div>
            <div style={{overflowX:"auto",display:"flex",gap:8,marginBottom:16,paddingBottom:2}}>
              {Object.entries(VOCAB).map(([key,v])=>(
                <Pill key={key} label={`${v.icon} ${v.label}`} active={fcSection===key} color={v.color}
                  onClick={()=>{setFcSection(key);setFcIdx(0);}}/>
              ))}
            </div>
            {fcItem && (
              <FlashCard
                front={fcItem.jp} back={[fcItem.romaji, fcItem.en, `e.g. ${fcItem.ex}`, `(${fcItem.exEn})`]}
                color={VOCAB[fcSection].color}
                idx={fcIdx} total={vocabWords.length}
                onPrev={()=>setFcIdx(i=>Math.max(0,i-1))}
                onNext={()=>setFcIdx(i=>Math.min(vocabWords.length-1,i+1))}
                onMark={()=>{setVocabProgress(p=>({...p,[`${fcSection}_${fcIdx}`]:!p[`${fcSection}_${fcIdx}`]}));showToast("⭐ Marked!",VOCAB[fcSection].color);}}
                marked={!!vocabProgress[`${fcSection}_${fcIdx}`]}
              />
            )}
            {/* All words list */}
            <div style={{fontSize:14,fontWeight:700,margin:"20px 0 10px"}}>All Words</div>
            {vocabWords.map((w,i)=>(
              <div key={i} onClick={()=>setFcIdx(i)} style={{
                display:"flex",justifyContent:"space-between",alignItems:"center",
                padding:"10px 14px",borderRadius:12,marginBottom:6,cursor:"pointer",
                background: i===fcIdx?"linear-gradient(90deg,"+VOCAB[fcSection].color+"22,"+VOCAB[fcSection].color+"11)":"#fff",
                border:`1.5px solid ${i===fcIdx?VOCAB[fcSection].color+"66":"#f0f0f0"}`
              }}>
                <div style={{fontSize:18,fontWeight:700}}>{w.jp}</div>
                <div style={{textAlign:"right"}}>
                  <div style={{fontSize:13,color:"#888"}}>{w.romaji}</div>
                  <div style={{fontSize:12,color:"#aaa"}}>{w.en}</div>
                </div>
              </div>
            ))}
          </div>
        )}

        {/* ── KANJI ── */}
        {tab==="kanji" && (
          <KanjiLearn progress={kanjiProgress} onMark={markKanji}/>
        )}

        {/* ── QUIZ / GAMES ── */}
        {tab==="quiz" && (
          <div>
            {!gameMode && (
              <div>
                <div style={{fontSize:14,fontWeight:700,marginBottom:10}}>Choose a Game Mode</div>
                {[
                  {id:"quiz",icon:"📝",label:"MCQ Quiz",sub:"Multiple choice questions",color:"#E84393"},
                  {id:"type",icon:"⌨️",label:"Type Quiz",sub:"Type the romaji yourself",color:"#7C3AED"},
                  {id:"listening",icon:"👂",label:"Listening",sub:"Hear & identify the character",color:"#10B981"},
                  {id:"speed",icon:"⚡",label:"Speed Round",sub:"3 seconds per question!",color:"#F59E0B"},
                  {id:"sentence",icon:"🧩",label:"Sentence Builder",sub:"Arrange words in order",color:"#6366F1"},
                  {id:"tongue",icon:"👅",label:"Tongue Twisters",sub:"Practice pronunciation",color:"#EC4899"},
                  {id:"grammar",icon:"📐",label:"Grammar Guide",sub:"Learn key grammar patterns",color:"#0EA5E9"},
                ].map(m=>(
                  <div key={m.id} onClick={()=>{sfx("click");setGameMode(m.id);}} style={{
                    display:"flex",alignItems:"center",gap:14,background:"#fff",borderRadius:16,
                    padding:"14px 16px",marginBottom:10,cursor:"pointer",
                    border:`2px solid ${m.color}22`,boxShadow:"0 2px 8px rgba(0,0,0,0.05)"
                  }}>
                    <div style={{width:48,height:48,borderRadius:14,background:m.color+"22",display:"flex",alignItems:"center",justifyContent:"center",fontSize:22,flexShrink:0}}>{m.icon}</div>
                    <div style={{flex:1}}>
                      <div style={{fontSize:15,fontWeight:700}}>{m.label}</div>
                      <div style={{fontSize:12,color:"#888"}}>{m.sub}</div>
                    </div>
                    <div style={{fontSize:20,color:"#ccc"}}>›</div>
                  </div>
                ))}
              </div>
            )}

            {(gameMode==="quiz"||gameMode==="type") && !quizResult && (
              <div>
                <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:12}}>
                  <button onClick={()=>{sfx("click");setGameMode(null);}} style={{background:"#f5f5f5",border:"none",borderRadius:10,padding:"8px 14px",cursor:"pointer",fontSize:13}}>← Back</button>
                  <div style={{fontSize:13,fontWeight:700,color:"#888"}}>
                    {gameMode==="quiz"?"MCQ Quiz":"Type Quiz"}
                  </div>
                </div>
                <div style={{display:"flex",gap:6,marginBottom:14,flexWrap:"wrap"}}>
                  {["hiragana","katakana","mixed","vocab"].map(s=>(
                    <Pill key={s} label={s.charAt(0).toUpperCase()+s.slice(1)} active={quizSection===s} color="#E84393" onClick={()=>{setQuizSection(s);setQuizKey(k=>k+1);}}/>
                  ))}
                </div>
                <Card>
                  <QuizEngine key={quizKey} pool={getQuizPool()} mode={gameMode}
                    onDone={r=>{setQuizResult(r);setTotalQuizzes(t=>t+r.total);setTotalCorrect(t=>t+r.score);}}/>
                </Card>
              </div>
            )}

            {(gameMode==="quiz"||gameMode==="type") && quizResult && (
              <Card>
                <QuizResult result={quizResult}
                  onRetry={()=>{setQuizResult(null);setQuizKey(k=>k+1);}}
                  onHome={()=>{setQuizResult(null);setGameMode(null);}}/>
              </Card>
            )}

            {gameMode==="listening" && !quizResult && (
              <div>
                <div style={{display:"flex",alignItems:"center",gap:10,marginBottom:14}}>
                  <button onClick={()=>{sfx("click");setGameMode(null);}} style={{background:"#f5f5f5",border:"none",borderRadius:10,padding:"8px 14px",cursor:"pointer"}}>← Back</button>
                  <div style={{fontSize:14,fontWeight:700}}>👂 Listening Practice</div>
                </div>
                <Card>
                  <ListeningGame pool={[...HIRAGANA,...KATAKANA]} onDone={r=>{setQuizResult(r);setTotalQuizzes(t=>t+r.total);setTotalCorrect(t=>t+r.score);}}/>
                </Card>
              </div>
            )}

            {gameMode==="listening" && quizResult && (
              <Card>
                <div style={{textAlign:"center",padding:20}}>
                  <div style={{fontSize:48,marginBottom:8}}>👂</div>
                  <div style={{fontSize:32,fontWeight:800,color:"#10B981",marginBottom:4}}>{quizResult.score}/{quizResult.total}</div>
                  <div style={{color:"#666",marginBottom:20}}>Listening score!</div>
                  <button onClick={()=>{sfx("click");setQuizResult(null);setGameMode(null);}} style={{padding:"12px 32px",borderRadius:14,background:"linear-gradient(90deg,#E84393,#7C3AED)",color:"#fff",border:"none",fontSize:15,fontWeight:700,cursor:"pointer"}}>Done</button>
                </div>
              </Card>
            )}

            {gameMode==="speed" && !quizResult && (
              <div>
                <div style={{display:"flex",alignItems:"center",gap:10,marginBottom:14}}>
                  <button onClick={()=>{sfx("click");setGameMode(null);}} style={{background:"#f5f5f5",border:"none",borderRadius:10,padding:"8px 14px",cursor:"pointer"}}>← Back</button>
                  <div style={{fontSize:14,fontWeight:700}}>⚡ Speed Round</div>
                </div>
                <Card>
                  <SpeedRound pool={[...HIRAGANA,...KATAKANA]} onDone={r=>{setQuizResult(r);setTotalQuizzes(t=>t+r.total);setTotalCorrect(t=>t+r.score);}}/>
                </Card>
              </div>
            )}

            {gameMode==="speed" && quizResult && (
              <Card>
                <div style={{textAlign:"center",padding:20}}>
                  <div style={{fontSize:48}}>⚡</div>
                  <div style={{fontSize:32,fontWeight:800,color:"#F59E0B",marginBottom:4}}>{quizResult.score}/{quizResult.total}</div>
                  <div style={{color:"#666",marginBottom:20}}>Speed score!</div>
                  <button onClick={()=>{sfx("click");setQuizResult(null);setGameMode(null);}} style={{padding:"12px 32px",borderRadius:14,background:"linear-gradient(90deg,#E84393,#7C3AED)",color:"#fff",border:"none",fontSize:15,fontWeight:700,cursor:"pointer"}}>Done</button>
                </div>
              </Card>
            )}

            {gameMode==="sentence" && (
              <div>
                <div style={{display:"flex",alignItems:"center",gap:10,marginBottom:14}}>
                  <button onClick={()=>{sfx("click");setGameMode(null);}} style={{background:"#f5f5f5",border:"none",borderRadius:10,padding:"8px 14px",cursor:"pointer"}}>← Back</button>
                  <div style={{fontSize:14,fontWeight:700}}>🧩 Sentence Builder</div>
                </div>
                <SentenceBuilder/>
              </div>
            )}

            {gameMode==="tongue" && (
              <div>
                <div style={{display:"flex",alignItems:"center",gap:10,marginBottom:14}}>
                  <button onClick={()=>{sfx("click");setGameMode(null);}} style={{background:"#f5f5f5",border:"none",borderRadius:10,padding:"8px 14px",cursor:"pointer"}}>← Back</button>
                  <div style={{fontSize:14,fontWeight:700}}>👅 Tongue Twisters</div>
                </div>
                <TongueTwisters/>
              </div>
            )}

            {gameMode==="grammar" && (
              <div>
                <div style={{display:"flex",alignItems:"center",gap:10,marginBottom:14}}>
                  <button onClick={()=>{sfx("click");setGameMode(null);}} style={{background:"#f5f5f5",border:"none",borderRadius:10,padding:"8px 14px",cursor:"pointer"}}>← Back</button>
                  <div style={{fontSize:14,fontWeight:700}}>📐 Grammar Guide</div>
                </div>
                {GRAMMAR.map((g,i)=>(
                  <Card key={i}>
                    <div style={{display:"flex",justifyContent:"space-between",alignItems:"flex-start",marginBottom:8}}>
                      <div style={{fontSize:15,fontWeight:800,color:"#E84393"}}>{g.title}</div>
                      <Badge label={g.level} color="#7C3AED"/>
                    </div>
                    <div style={{background:"#f8f8f8",borderRadius:10,padding:"8px 12px",fontFamily:"monospace",fontSize:13,color:"#555",marginBottom:8}}>
                      {g.pattern}
                    </div>
                    <div style={{fontSize:13,color:"#666",marginBottom:10}}>{g.en}</div>
                    {g.examples.map((ex,j)=>(
                      <div key={j} style={{borderLeft:"3px solid #E84393",paddingLeft:10,marginBottom:8}}>
                        <div style={{fontSize:14,fontWeight:600,cursor:"pointer"}} onClick={()=>{sfx("click");speak(ex.jp);}}>{ex.jp} 🔊</div>
                        <div style={{fontSize:12,color:"#888"}}>{ex.en}</div>
                      </div>
                    ))}
                    <div style={{background:"#fff8f0",borderRadius:10,padding:"8px 12px",marginTop:6}}>
                      <span style={{fontSize:12,color:"#F59E0B",fontWeight:700}}>💡 Tip: </span>
                      <span style={{fontSize:12,color:"#555"}}>{g.tip}</span>
                    </div>
                  </Card>
                ))}
              </div>
            )}
          </div>
        )}

        {/* ── AI TUTOR ── */}
        {tab==="ai" && (
          <Card style={{height:"calc(100vh - 200px)",display:"flex",flexDirection:"column"}}>
            <AITutor/>
          </Card>
        )}

        {/* ── MORE ── */}
        {tab==="more" && (
          <div>
            <div style={{fontSize:14,fontWeight:700,marginBottom:12}}>Sound Settings 🔊</div>
            <SoundSettingsPanel/>

            <div style={{fontSize:14,fontWeight:700,marginBottom:12,marginTop:18}}>More Features</div>
            {[
              {icon:"🗾",label:"Culture Notes",sub:"6 fascinating Japanese culture facts"},
              {icon:"👅",label:"Tongue Twisters",sub:"Perfect your pronunciation"},
              {icon:"📐",label:"Grammar Patterns",sub:"8 essential N5 grammar rules"},
              {icon:"🧩",label:"Sentence Builder",sub:"Arrange words to form sentences"},
            ].map((item,i)=>(
              <Card key={i} style={{display:"flex",alignItems:"center",gap:12,cursor:"pointer"}}
                onClick={()=>{
                  sfx("click");
                  setTab("quiz");
                  setGameMode(item.label.includes("Culture")?"grammar":item.label.includes("Tongue")?"tongue":item.label.includes("Grammar")?"grammar":"sentence");
                }}>
                <div style={{fontSize:30}}>{item.icon}</div>
                <div>
                  <div style={{fontSize:14,fontWeight:700}}>{item.label}</div>
                  <div style={{fontSize:12,color:"#888"}}>{item.sub}</div>
                </div>
                <div style={{marginLeft:"auto",color:"#ccc",fontSize:20}}>›</div>
              </Card>
            ))}
            <Card style={{background:"linear-gradient(135deg,#fff5fa,#f5f0ff)",border:"2px solid #E84393"}}>
              <div style={{fontSize:13,color:"#E84393",fontWeight:700,marginBottom:6}}>🎌 JLPT Roadmap</div>
              {[
                {level:"N5",desc:"Hiragana, Katakana, ~100 Kanji, basic grammar",color:"#10B981"},
                {level:"N4",desc:"~300 Kanji, verb conjugation, compound sentences",color:"#3B82F6"},
                {level:"N3",desc:"~650 Kanji, complex grammar, newspapers",color:"#F59E0B"},
                {level:"N2",desc:"~1000 Kanji, near-native comprehension",color:"#E84393"},
                {level:"N1",desc:"~2000 Kanji, native-level fluency",color:"#7C3AED"},
              ].map((l,i)=>(
                <div key={i} style={{display:"flex",gap:10,alignItems:"center",marginBottom:8}}>
                  <Badge label={l.level} color={l.color}/>
                  <span style={{fontSize:12,color:"#555"}}>{l.desc}</span>
                </div>
              ))}
              <div style={{fontSize:11,color:"#aaa",marginTop:4}}>You are here: N5 beginner 🎯</div>
            </Card>
          </div>
        )}
      </div>

      {/* BOTTOM NAV */}
      <div style={{
        position:"fixed",bottom:0,left:"50%",transform:"translateX(-50%)",
        width:"100%",maxWidth:480,background:"#fff",
        borderTop:"1px solid #eee",display:"flex",zIndex:100,
        paddingBottom:"env(safe-area-inset-bottom)"
      }}>
        {TABS.map(t=>(
          <button key={t.id} onClick={()=>{if(tab!==t.id) sfx("click"); setTab(t.id);setGameMode(null);setQuizResult(null);}} style={{
            flex:1,display:"flex",flexDirection:"column",alignItems:"center",gap:2,
            background:"none",border:"none",cursor:"pointer",padding:"8px 0 6px",
            color: tab===t.id?"#E84393":"#aaa",transition:"color 0.2s"
          }}>
            <span style={{fontSize:18,lineHeight:1}}>{t.icon}</span>
            <span style={{fontSize:9,fontWeight:tab===t.id?700:400,letterSpacing:.3}}>{t.label}</span>
            {tab===t.id && <div style={{width:16,height:2.5,borderRadius:2,background:"#E84393",marginTop:1}}/>}
          </button>
        ))}
      </div>
    </div>
  );
}
