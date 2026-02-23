export default {
  async fetch(request, env) {
    // Health check
    if (request.method === "GET") return new Response("ok", { status: 200 });
    if (request.method !== "POST") return new Response("Method not allowed", { status: 405 });

    // --- Required env vars ---
    const BOT_TOKEN = env.BOT_TOKEN;
    if (!BOT_TOKEN) return new Response("Missing BOT_TOKEN", { status: 500 });

    const ADMIN_CHAT_ID = Number(env.ADMIN_CHAT_ID || 0);
    const TG_API = `https://api.telegram.org/bot${BOT_TOKEN}`;

    // --- Telegram helper ---
    const tg = async (method, payload) => {
      const r = await fetch(`${TG_API}/${method}`, {
        method: "POST",
        headers: { "content-type": "application/json" },
        body: JSON.stringify(payload),
      });
      return r.json();
    };

    // --- Keyboards ---
    const replyKeyboard = (rows) => ({
      keyboard: rows.map((row) => row.map((t) => ({ text: t }))),
      resize_keyboard: true,
      one_time_keyboard: true,
    });
    const removeKeyboard = () => ({ remove_keyboard: true });
    const inlineKeyboard = (rows) => ({ inline_keyboard: rows });

    // --- KV state helpers ---
    const getState = async (uid) => {
      const raw = await env.STATE?.get(String(uid));
      if (!raw) return null;
      try { return JSON.parse(raw); } catch { return null; }
    };
    const setState = async (uid, st) => env.STATE.put(String(uid), JSON.stringify(st));
    const clearState = async (uid) => env.STATE.delete(String(uid));

    // --- Copy / Options (Persian) ---
    const INTRO =
      "سلام 👋✨\n" +
      "به فرم بررسی اولیه اپلای تحصیلی <b>Negzi English</b> خوش اومدی 🎓🌍\n\n" +
      "چند سوال کوتاه می‌پرسیم (حدود ۳–۴ دقیقه ⏱️) و بعدش برای شما یک <b>جلسه ۱۵ دقیقه‌ای مشاوره رایگان</b> هماهنگ می‌کنیم ✅\n\n" +
      "⚠️ ظرفیت روزانه محدوده و این آفر فقط تا پایان هفته فعاله ⏳🔥\n\n" +
      "آماده‌ای شروع کنیم؟ 😄";

    const Q = {
      1: "1️⃣🎯 برای چه <b>مقطع</b>ی می‌خوای اپلای کنی؟",
      2: "2️⃣📚 رشته تحصیلی فعلی یا آخرین رشته‌ای که خوندی چی بوده؟",
      3: "3️⃣🏫 دانشگاه محل تحصیل؟\n(اسم دانشگاه + کشور 🌍)",
      4: "4️⃣🧮 معدل همه مقاطع رو وارد کن.\n(دیپلم، لیسانس، ارشد، دکتری اگر داری)",
      5: "5️⃣🗣️ وضعیت زبان انگلیسی‌ات کدومه؟",
      6:
        "✍️ اگر آزمون دادی، <b>نمره + تاریخ</b> رو بنویس.\n" +
        "مثال: IELTS 7.0 – Sep 2024\n\n" +
        "اگر آزمون ندادی، سطح فعلیت رو حدودی بگو (مثلاً B2 / Intermediate).",
      7:
        "6️⃣🌍 برای کدوم کشورها قصد اپلای داری؟ (می‌تونی چندتا انتخاب کنی)\n" +
        "آخرش «✅ تمام» رو بزن.",
      8: "7️⃣💼 سابقه کاری مرتبط داری؟",
      9: "📝 سابقه کار رو با <b>زمان + عنوان شغلی</b> خیلی کوتاه بنویس.",
      10: "8️⃣🔬 سابقه مقاله/پژوهش داری؟",
      11: "🧪 خیلی خلاصه سابقه پژوهشیت رو با ذکر زمان توضیح بده.",
      12: "📄 اگر مقاله داری، موضوع/حوزه‌اش رو خیلی کوتاه بنویس.",
      13: "9️⃣✉️ تا حالا با استاد برای اپلای مکاتبه داشتی؟",
      14:
        "🔟🎂 سن فعلی و گپ تحصیلی (اگر هست) رو بنویس.\n" +
        "مثال: 27 سال | 2 سال گپ",
      15: "1️⃣1️⃣⭐ هدف اصلی‌ات از اپلای چیه؟",
      16: "1️⃣2️⃣🤔 مهم‌ترین سوال/نگرانی‌ات درباره اپلای چیه؟",
    };

    const DONE =
      "✅ <b>ثبت شد!</b> ممنون از وقتی که گذاشتی 🙌✨\n\n" +
      "تیم ما یک <b>جلسه ۱۵ دقیقه‌ای مشاوره رایگان</b> برات هماهنگ می‌کنه 🧭📞\n" +
      "⏳ ظرفیت محدوده و اولویت با فرم‌های کامل و دقیق است.";

    const OPT = {
      degree: ["کارشناسی", "ارشد (Course-based)", "ارشد (Research / Thesis)", "دکتری (PhD)", "هنوز مطمئن نیستم 🤷"],
      lang: ["آیلتس دارم ✅", "تافل دارم ✅", "هنوز آزمون ندادم", "در حال آماده‌سازی هستم 📖"],
      work: ["ندارم", "کمتر از ۱ سال", "۱ تا ۳ سال", "بیشتر از ۳ سال"],
      research: ["ندارم", "مقاله داخلی", "مقاله ISI / Scopus", "در حال آماده‌سازی ✍️"],
      prof: ["بله، پاسخ گرفتم ✅", "مکاتبه کردم ولی جواب نگرفتم", "هنوز مکاتبه نکردم", "برای مقطع من لازم نیست", "اصلاً نمی‌دونم مکاتبه چیه 😅"],
      goal: ["فول فاند 💰", "پذیرش بدون فاند", "مهاجرت تحصیلی", "هنوز در حال بررسی‌ام 🤔"],
      countries: ["کانادا 🇨🇦", "آمریکا 🇺🇸", "اروپا 🇪🇺", "سایر (تایپ کن) ✍️", "هنوز تصمیم نگرفتم 🤷"],
    };

    const mustBeOneOf = (value, options) => options.includes(value);

    // --- Country inline keyboard ---
    const buildCountryKeyboard = (selected) => {
      const rows = [];
      for (let i = 0; i < OPT.countries.length; i += 2) {
        const pair = OPT.countries.slice(i, i + 2).map((c) => ({
          text: `${selected.includes(c) ? "✅" : "⬜️"} ${c}`,
          callback_data: `country:${c}`,
        }));
        rows.push(pair);
      }
      rows.push([{ text: "✅ تمام", callback_data: "country_done" }]);
      return inlineKeyboard(rows);
    };

    // --- Ask next step ---
    const askStep = async (chatId, st) => {
      const step = st.step;

      if (step === 1) {
        await tg("sendMessage", {
          chat_id: chatId,
          text: INTRO,
          parse_mode: "HTML",
          reply_markup: removeKeyboard(),
        });
        await tg("sendMessage", {
          chat_id: chatId,
          text: Q[1],
          parse_mode: "HTML",
          reply_markup: replyKeyboard([OPT.degree]),
        });
        return;
      }

      if (step === 2) return tg("sendMessage", { chat_id: chatId, text: Q[2], parse_mode: "HTML", reply_markup: removeKeyboard() });
      if (step === 3) return tg("sendMessage", { chat_id: chatId, text: Q[3], parse_mode: "HTML", reply_markup: removeKeyboard() });
      if (step === 4) return tg("sendMessage", { chat_id: chatId, text: Q[4], parse_mode: "HTML", reply_markup: removeKeyboard() });

      if (step === 5) {
        return tg("sendMessage", {
          chat_id: chatId,
          text: Q[5],
          parse_mode: "HTML",
          reply_markup: replyKeyboard([OPT.lang]),
        });
      }

      if (step === 6) return tg("sendMessage", { chat_id: chatId, text: Q[6], parse_mode: "HTML", reply_markup: removeKeyboard() });

      if (step === 7) {
        st.countrySet = [];
        st.awaitingOtherCountry = false;

        await tg("sendMessage", { chat_id: chatId, text: Q[7], parse_mode: "HTML", reply_markup: removeKeyboard() });

        const m = await tg("sendMessage", {
          chat_id: chatId,
          text: "👇 انتخاب کن:",
          parse_mode: "HTML",
          reply_markup: buildCountryKeyboard(st.countrySet),
        });

        st.countryMsgId = m?.result?.message_id ?? null;
        return;
      }

      if (step === 8) return tg("sendMessage", { chat_id: chatId, text: Q[8], parse_mode: "HTML", reply_markup: replyKeyboard([OPT.work]) });
      if (step === 9) return tg("sendMessage", { chat_id: chatId, text: Q[9], parse_mode: "HTML", reply_markup: removeKeyboard() });

      if (step === 10) return tg("sendMessage", { chat_id: chatId, text: Q[10], parse_mode: "HTML", reply_markup: replyKeyboard([OPT.research]) });
      if (step === 11) return tg("sendMessage", { chat_id: chatId, text: Q[11], parse_mode: "HTML", reply_markup: removeKeyboard() });
      if (step === 12) return tg("sendMessage", { chat_id: chatId, text: Q[12], parse_mode: "HTML", reply_markup: removeKeyboard() });

      if (step === 13) return tg("sendMessage", { chat_id: chatId, text: Q[13], parse_mode: "HTML", reply_markup: replyKeyboard([OPT.prof]) });
      if (step === 14) return tg("sendMessage", { chat_id: chatId, text: Q[14], parse_mode: "HTML", reply_markup: removeKeyboard() });

      if (step === 15) return tg("sendMessage", { chat_id: chatId, text: Q[15], parse_mode: "HTML", reply_markup: replyKeyboard([OPT.goal]) });
      if (step === 16) return tg("sendMessage", { chat_id: chatId, text: Q[16], parse_mode: "HTML", reply_markup: removeKeyboard() });

      if (step === 17) {
        await tg("sendMessage", { chat_id: chatId, text: DONE, parse_mode: "HTML", reply_markup: removeKeyboard() });

        if (ADMIN_CHAT_ID) {
          const a = st.answers || {};
          const summary =
            "📩 <b>فرم جدید اپلای</b>\n\n" +
            `1) 🎯 مقطع: ${a.degree || "-"}\n` +
            `2) 📚 رشته: ${a.major || "-"}\n` +
            `3) 🏫 دانشگاه: ${a.uni || "-"}\n` +
            `4) 🧮 معدل‌ها: ${a.gpa || "-"}\n` +
            `5) 🗣️ زبان: ${a.lang_status || "-"} | ${a.lang_detail || "-"}\n` +
            `6) 🌍 کشورها: ${a.countries || "-"}\n` +
            `7) 💼 سابقه کار: ${a.work_years || "-"} | ${a.work_detail || "-"}\n` +
            `8) 🔬 پژوهش: ${a.research_level || "-"} | ${a.research_detail || "-"} | ${a.paper_topic || "-"}\n` +
            `9) ✉️ مکاتبه استاد: ${a.prof_contact || "-"}\n` +
            `10) 🎂 سن/گپ: ${a.age_gap || "-"}\n` +
            `11) ⭐ هدف: ${a.goal || "-"}\n` +
            `12) 🤔 نگرانی: ${a.concern || "-"}\n`;

          await tg("sendMessage", { chat_id: ADMIN_CHAT_ID, text: summary, parse_mode: "HTML" });
        }

        await clearState(st.uid);
        return;
      }
    };

    // --- Parse update ---
    const update = await request.json();

    // 1) Handle country callback buttons
    if (update.callback_query) {
      const cq = update.callback_query;
      const uid = cq.from.id;
      const chatId = cq.message.chat.id;

      let st = await getState(uid);
      if (!st) st = { uid, step: 0, answers: {}, countrySet: [], awaitingOtherCountry: false, countryMsgId: null };

      const data = cq.data || "";

      if (st.step !== 7) {
        await tg("answerCallbackQuery", { callback_query_id: cq.id, text: "این مرحله فعال نیست." });
        return new Response("ok");
      }

      if (data.startsWith("country:")) {
        const c = data.slice("country:".length);

        if (c === "هنوز تصمیم نگرفتم 🤷") {
          st.countrySet = [c];
        } else {
          st.countrySet = st.countrySet.filter((x) => x !== "هنوز تصمیم نگرفتم 🤷");
          st.countrySet = st.countrySet.includes(c) ? st.countrySet.filter((x) => x !== c) : [...st.countrySet, c];
        }

        await setState(uid, st);
        await tg("answerCallbackQuery", { callback_query_id: cq.id });

        await tg("editMessageReplyMarkup", {
          chat_id: chatId,
          message_id: cq.message.message_id,
          reply_markup: buildCountryKeyboard(st.countrySet),
        });

        return new Response("ok");
      }

      if (data === "country_done") {
        if (!st.countrySet.length) {
          await tg("answerCallbackQuery", { callback_query_id: cq.id, text: "حداقل یک گزینه انتخاب کن 🙂" });
          return new Response("ok");
        }

        if (st.countrySet.includes("سایر (تایپ کن) ✍️")) {
          st.awaitingOtherCountry = true;
          await setState(uid, st);
          await tg("answerCallbackQuery", { callback_query_id: cq.id, text: "اوکی ✅" });
          await tg("sendMessage", { chat_id: chatId, text: "✍️ کشور(های) دیگر رو تایپ کن (مثلاً: استرالیا، هلند):", parse_mode: "HTML" });
          return new Response("ok");
        }

        st.answers.countries = st.countrySet.filter((x) => x !== "سایر (تایپ کن) ✍️").join("، ");
        st.step = 8;

        await setState(uid, st);

        // Optional: remove inline keyboard after done
        if (cq.message?.message_id) {
          await tg("editMessageReplyMarkup", { chat_id: chatId, message_id: cq.message.message_id, reply_markup: {} });
        }

        await tg("answerCallbackQuery", { callback_query_id: cq.id, text: "ثبت شد ✅" });
        await askStep(chatId, st);
        await setState(uid, st);

        return new Response("ok");
      }

      await tg("answerCallbackQuery", { callback_query_id: cq.id });
      return new Response("ok");
    }

    // 2) Handle normal messages
    const msg = update.message;
    if (!msg || !msg.from) return new Response("ok");

    const uid = msg.from.id;
    const chatId = msg.chat.id;
    const text = (msg.text || "").trim();

    let st = await getState(uid);
    if (!st) st = { uid, step: 0, answers: {}, countrySet: [], awaitingOtherCountry: false, countryMsgId: null };

    // /start payload support: "/start apply"
    if (text.startsWith("/start") || text.startsWith("/restart")) {
      st = { uid, step: 1, answers: {}, countrySet: [], awaitingOtherCountry: false, countryMsgId: null };
      await setState(uid, st);
      await askStep(chatId, st);
      await setState(uid, st);
      return new Response("ok");
    }

    // If user is typing other countries
    if (st.step === 7 && st.awaitingOtherCountry) {
      const selected = st.countrySet.filter((x) => x !== "سایر (تایپ کن) ✍️");
      const combined = [...selected, ...(text ? [text] : [])].filter(Boolean);
      st.answers.countries = combined.join("، ");
      st.awaitingOtherCountry = false;
      st.step = 8;

      await setState(uid, st);
      await askStep(chatId, st);
      await setState(uid, st);
      return new Response("ok");
    }

    if (st.step === 0) {
      await tg("sendMessage", { chat_id: chatId, text: "برای شروع فرم، /start رو بزن 🙂" });
      return new Response("ok");
    }

    // Step validations + saves
    if (st.step === 1) {
      if (!mustBeOneOf(text, OPT.degree)) {
        await tg("sendMessage", {
          chat_id: chatId,
          text: "لطفاً یکی از گزینه‌ها رو انتخاب کن 👇",
          reply_markup: replyKeyboard([OPT.degree]),
        });
        return new Response("ok");
      }
      st.answers.degree = text;
      st.step = 2;
    } else if (st.step === 2) {
      st.answers.major = text;
      st.step = 3;
    } else if (st.step === 3) {
      st.answers.uni = text;
      st.step = 4;
    } else if (st.step === 4) {
      st.answers.gpa = text;
      st.step = 5;
    } else if (st.step === 5) {
      if (!mustBeOneOf(text, OPT.lang)) {
        await tg("sendMessage", {
          chat_id: chatId,
          text: "یکی از گزینه‌ها رو انتخاب کن 👇",
          reply_markup: replyKeyboard([OPT.lang]),
        });
        return new Response("ok");
      }
      st.answers.lang_status = text;
      st.step = 6;
    } else if (st.step === 6) {
      st.answers.lang_detail = text;
      st.step = 7;
    } else if (st.step === 7) {
      await tg("sendMessage", { chat_id: chatId, text: "کشورها رو با دکمه‌ها انتخاب کن و «✅ تمام» رو بزن 🙂" });
      await setState(uid, st);
      return new Response("ok");
    } else if (st.step === 8) {
      if (!mustBeOneOf(text, OPT.work)) {
        await tg("sendMessage", { chat_id: chatId, text: "یکی از گزینه‌ها رو انتخاب کن 👇", reply_markup: replyKeyboard([OPT.work]) });
        return new Response("ok");
      }
      st.answers.work_years = text;
      st.step = 9;
    } else if (st.step === 9) {
      st.answers.work_detail = text;
      st.step = 10;
    } else if (st.step === 10) {
      if (!mustBeOneOf(text, OPT.research)) {
        await tg("sendMessage", { chat_id: chatId, text: "یکی از گزینه‌ها رو انتخاب کن 👇", reply_markup: replyKeyboard([OPT.research]) });
        return new Response("ok");
      }
      st.answers.research_level = text;
      st.step = 11;
    } else if (st.step === 11) {
      st.answers.research_detail = text;
      st.step = 12;
    } else if (st.step === 12) {
      st.answers.paper_topic = text;
      st.step = 13;
    } else if (st.step === 13) {
      if (!mustBeOneOf(text, OPT.prof)) {
        await tg("sendMessage", { chat_id: chatId, text: "یکی از گزینه‌ها رو انتخاب کن 👇", reply_markup: replyKeyboard([OPT.prof]) });
        return new Response("ok");
      }
      st.answers.prof_contact = text;
      st.step = 14;
    } else if (st.step === 14) {
      st.answers.age_gap = text;
      st.step = 15;
    } else if (st.step === 15) {
      if (!mustBeOneOf(text, OPT.goal)) {
        await tg("sendMessage", { chat_id: chatId, text: "یکی از گزینه‌ها رو انتخاب کن 👇", reply_markup: replyKeyboard([OPT.goal]) });
        return new Response("ok");
      }
      st.answers.goal = text;
      st.step = 16;
    } else if (st.step === 16) {
      st.answers.concern = text;
      st.step = 17;
    }

    await setState(uid, st);
    await askStep(chatId, st);
    await setState(uid, st);

    return new Response("ok");
  },
};
