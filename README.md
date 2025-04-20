# Omad_charxi_bot.py
import logging import random from aiogram import Bot, Dispatcher, executor, types from aiogram.types import InlineKeyboardMarkup, InlineKeyboardButton from aiogram.contrib.fsm_storage.memory import MemoryStorage from aiogram.dispatcher import FSMContext from aiogram.dispatcher.filters.state import State, StatesGroup

API_TOKEN = '7703457334:AAGxox3Q6--n7cnOmNVN0a01lYbxXZeX3DY' CHANNEL_USERNAME = "@smartlife_samarqand" ADMIN_ID = 7602284509

logging.basicConfig(level=logging.INFO)

bot = Bot(token=API_TOKEN) dp = Dispatcher(bot, storage=MemoryStorage())

users = {} sovgalar = ["1000 so'm", "500 MB", "Qayta urin", "Hech narsa chiqmadi"] purchases = set()

class AddSovga(StatesGroup): waiting_for_sovga = State()

Kanalga obuna tekshirish

async def is_subscribed(user_id): try: member = await bot.get_chat_member(CHANNEL_USERNAME, user_id) return member.status in ["member", "creator", "administrator"] except: return False

Start

@dp.message_handler(commands=['start']) async def start_handler(message: types.Message): user_id = message.from_user.id ref_id = message.get_args()

if ref_id and ref_id != str(user_id):
    if user_id not in users:
        users[user_id] = {"ref": int(ref_id), "invited": 0, "spins": 0}
        if int(ref_id) in users:
            users[int(ref_id)]["invited"] += 1
elif user_id not in users:
    users[user_id] = {"ref": None, "invited": 0, "spins": 0}

sub = await is_subscribed(user_id)
if not sub:
    btn = InlineKeyboardMarkup().add(InlineKeyboardButton("Kanalga obuna bo'lish", url=f"https://t.me/{CHANNEL_USERNAME[1:]}")).add(InlineKeyboardButton("Tekshirish", callback_data="check_sub"))
    await message.answer("Iltimos, botdan foydalanish uchun kanalga obuna bo'ling:", reply_markup=btn)
else:
    await message.answer("Barabanni aylantirish uchun 10 ta do'stingizni taklif qiling yoki biror mahsulot xarid qiling. Sizning referal linkingiz:")
    await message.answer(f"https://t.me/{(await bot.get_me()).username}?start={user_id}")

Obuna tekshirish

@dp.callback_query_handler(lambda c: c.data == 'check_sub') async def check_sub_callback(callback_query: types.CallbackQuery): user_id = callback_query.from_user.id sub = await is_subscribed(user_id) if sub: await bot.answer_callback_query(callback_query.id, "Obuna bo'ldingiz!") await bot.send_message(user_id, "Endi barabanni aylantirish uchun 10 ta do'st taklif qiling yoki mahsulot xarid qiling.") else: await bot.answer_callback_query(callback_query.id, "Hali ham obuna emassiz!")

Baraban komandasi

@dp.message_handler(commands=['baraban']) async def spin_baraban(message: types.Message): user_id = message.from_user.id user = users.get(user_id, {"invited": 0, "spins": 0})

if user["invited"] >= 10 or user_id in purchases:
    result = random.choice(sovgalar)
    users[user_id]["spins"] = 0
    purchases.discard(user_id)
    await message.answer(f"Baraban aylanmoqda...\nSovrin: {result}")
else:
    await message.answer("Siz hali 10 ta do‘st taklif qilmagansiz yoki mahsulot xarid qilmagansiz.")

Admin: Sovg'a qo‘shish

@dp.message_handler(commands=['addsovga']) async def add_sovga_cmd(message: types.Message): if message.from_user.id == ADMIN_ID: await message.answer("Yangi sovg'ani yuboring:") await AddSovga.waiting_for_sovga.set()

@dp.message_handler(state=AddSovga.waiting_for_sovga) async def process_new_sovga(message: types.Message, state: FSMContext): sovgalar.append(message.text) await message.answer("Sovg'a qo‘shildi!") await state.finish()

Admin: Sovg'alar ro'yxati

@dp.message_handler(commands=['listsovga']) async def list_sovga(message: types.Message): if message.from_user.id == ADMIN_ID: text = "Sovg'alar ro'yxati:\n" + "\n".join(f"- {s}" for s in sovgalar) await message.answer(text)

Admin: Xarid qo‘shish

@dp.message_handler(commands=['addpurchase']) async def add_purchase(message: types.Message): if message.from_user.id == ADMIN_ID: try: parts = message.text.split() uid = int(parts[1]) purchases.add(uid) await message.answer("Xarid qo‘shildi!") except: await message.answer("Foydalanuvchi ID ni to‘g‘ri kiriting: /addpurchase 123456789")

if name == 'main': executor.start_polling(dp, skip_updates=True)

