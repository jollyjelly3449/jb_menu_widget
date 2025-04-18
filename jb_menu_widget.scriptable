const fm = FileManager.local();
const cachePath = fm.joinPath(fm.documentsDirectory(), "mealCache.json");
let isOffline = false;

// 🧹 HTML cell cleaner
function clean(cellHtml) {
  return cellHtml
    .replace(/<[^>]+>/g, "")
    .replace(/&nbsp;/g, " ")
    .replace(/<br\s*\/?>/gi, "\n")
    .trim();
}

// 🗓️ Date setup
const now = new Date();
let targetDate = new Date(now);

const hour = now.getHours();
const minute = now.getMinutes();
const totalMinutes = hour * 60 + minute;

// 🔁 Determine meal type
const breakfastEmojis = ["🥣", "🍞", "🥐"];
const lunchEmojis = ["🍛", "🍱", "🍜"];
const dinnerEmojis = ["🍲", "🍖", "🍚"];

function pick(arr) {
  return arr[Math.floor(Math.random() * arr.length)];
}

let mealLabel = "";
let mealIndex = 0;

if (totalMinutes >= 1170 || totalMinutes < 510) {
//  mealLabel = "🥣 내일 아침";
  mealLabel = `${pick(breakfastEmojis)} 내일 아침`;
  mealIndex = 0;
  targetDate.setDate(targetDate.getDate() + 1);
} else if (totalMinutes >= 510 && totalMinutes < 780) {
//  mealLabel = "🍛 점심";
  mealLabel = `${pick(breakfastEmojis)} 점심`;
  mealIndex = 1;
} else if (totalMinutes >= 780 && totalMinutes < 1170) {
//  mealLabel = "🍲 저녁";
  mealLabel = `${pick(breakfastEmojis)} 저녁`;
  mealIndex = 2;
} else {
//  mealLabel = "🥣 아침";
  mealLabel = `${pick(breakfastEmojis)} 아침`;
  mealIndex = 0;
}

const month = targetDate.getMonth() + 1;
const day = targetDate.getDate();
const dateStr = `${month}. ${day}`;
console.log(`🗓️ Targeting ${dateStr} for meal: ${mealLabel}`);

let html = "";
try {
  const req = new Request("https://seoul.jbiles.or.kr/bbs/board.php?bo_table=sub03_02");
  req.headers = {
    "User-Agent": "Mozilla/5.0 (iPhone; CPU iPhone OS 17_0 like Mac OS X)",
  };
  html = await req.loadString();
  console.log("✅ HTML loaded");
} catch (e) {
  console.log("❌ Failed to fetch HTML:", e);
  isOffline=true
}

let rows = [];
if (html) {
  rows = html.match(/<tr[\s\S]*?<\/tr>/gi) || [];

  // Try saving cache
  try {
    const parsed = rows.map(row => {
      const cells = [...row.matchAll(/<td[^>]*>([\s\S]*?)<\/td>/g)];
      if (cells.length >= 5) {
        const date = clean(cells[1][1]);
        return {
          date,
          breakfast: clean(cells[2][1]),
          lunch: clean(cells[3][1]),
          dinner: clean(cells[4][1]),
        };
      }
      return null;
    }).filter(Boolean);

    fm.writeString(cachePath, JSON.stringify(parsed));
    console.log("✅ Cache updated");
  } catch (e) {
    console.log("⚠️ Failed to update cache:", e);
  }
}

let targetRow = rows.find(row => row.includes(dateStr));
let content = "";

const isSunday = now.getDay() === 0;
const requestingTomorrowBreakfast = (mealIndex === 0 && totalMinutes >= 1170);

// Sunday fallback logic
if (!targetRow && isSunday && requestingTomorrowBreakfast) {
  const todayStr = `${now.getMonth() + 1}. ${now.getDate()}`;
  targetRow = rows.find(row => row.includes(todayStr));
  if (targetRow) {
    const cells = [...targetRow.matchAll(/<td[^>]*>([\s\S]*?)<\/td>/g)];
    if (cells.length >= 5) {
      const fallbackDinner = clean(cells[4][1]);
      content = "월요일 식단이 아직 등록되지 않았습니다.\n오늘 저녁을 대신 표시합니다.\n\n" + fallbackDinner;
      mealLabel = "🍲 오늘 저녁 (임시)";
      targetDate = now;
    }
  }
}

// Cache fallback if still no row
if (!targetRow && !content) {
  if (fm.fileExists(cachePath)) {
    const cache = JSON.parse(fm.readString(cachePath));
    const fallback = cache.find(row => row.date === dateStr);
    if (fallback) {
      content = [fallback.breakfast, fallback.lunch, fallback.dinner][mealIndex];
      console.log("📦 Loaded from cache");
    } else {
      content = "식단 정보를 불러올 수 없습니다.";
    }
  } else {
    content = "식단 정보를 불러올 수 없습니다 (캐시 없음)";
  }
}

// If found, extract menu
if (targetRow && !content) {
  const cells = [...targetRow.matchAll(/<td[^>]*>([\s\S]*?)<\/td>/g)];
  if (cells.length >= 5) {
    const breakfast = clean(cells[2][1]);
    const lunch = clean(cells[3][1]);
    const dinner = clean(cells[4][1]);
    content = [breakfast, lunch, dinner][mealIndex];
  } else {
    content = "식단 정보를 불러올 수 없습니다.";
  }
}

// 🧱 Build widget
const w = new ListWidget();
w.addText(`🗓️ ${month}/${day} 식단`).font = Font.boldSystemFont(16);
w.addSpacer(6);
// w.addText(mealLabel).font = Font.semiboldSystemFont(12);
// w.addText(content).font = Font.systemFont(10);
// w.addSpacer();

const mealTitle = w.addText(mealLabel);
mealTitle.font = Font.semiboldSystemFont(13);

w.addSpacer(2);

// Add dash to each line of the menu
const dashedContent = content
  .split("\n")
  .filter(line => line.trim() !== "")
  .map(line => `- ${line}`)
  .join("\n");

// const mealText = w.addText(content);
const mealText = w.addText(dashedContent)
mealText.font = Font.systemFont(11);
mealText.lineLimit = 0; // allow as many lines as needed
mealText.minimumScaleFactor = 0.8; // scale text down if needed

w.addSpacer();
w.addSpacer(4);

w.url = "https://seoul.jbiles.or.kr/bbs/board.php?bo_table=sub03_02";

if (isOffline) {
  w.addSpacer(4);
  const offlineText = w.addText("오프라인 모드");
  offlineText.font = Font.italicSystemFont(9);
  offlineText.textColor = Color.gray();
}

Script.setWidget(w);
Script.complete();
