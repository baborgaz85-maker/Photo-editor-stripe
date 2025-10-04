<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8">
  <title>محرر الصور - نسخة مدفوعة</title>
  <script src="https://js.stripe.com/v3/"></script>
  <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
</head>
<body class="bg-gray-100 font-sans">

<header class="bg-blue-600 text-white p-6 text-center">
  <h1 class="text-3xl font-bold">✦ محرر الصور الاحترافي ✦</h1>
  <p class="mt-2">عدل صورك بسهولة — مع ميزات مدفوعة</p>
</header>

<section class="text-center py-10">
  <h2 class="text-2xl font-bold mb-6">خطط الأسعار</h2>
  <div class="flex flex-col md:flex-row justify-center gap-8 max-w-5xl mx-auto">

    <div class="bg-white p-8 rounded-lg shadow-md flex-1 text-center">
      <h3 class="text-xl font-bold">الخطة المجانية</h3>
      <p class="text-3xl font-bold my-4">0$</p>
      <ul class="text-gray-600 mb-6">
        <li>✦ قص الصور</li>
        <li>✦ تغيير الحجم</li>
      </ul>
      <button class="bg-gray-400 text-white px-6 py-2 rounded-lg cursor-not-allowed">مجانية</button>
    </div>

    <div class="bg-blue-600 text-white p-8 rounded-lg shadow-md flex-1 text-center">
      <h3 class="text-xl font-bold">الخطة الاحترافية</h3>
      <p class="text-3xl font-bold my-4">5$/شهر</p>
      <ul class="mb-6">
        <li>✦ كل ميزات الخطة المجانية</li>
        <li>✦ فلاتر متقدمة</li>
        <li>✦ تحسين جودة الصور</li>
        <li>✦ تحميل غير محدود</li>
      </ul>
      <button id="checkout-button" class="bg-white text-blue-600 font-bold px-6 py-2 rounded-lg">
        اشترك الآن
      </button>
    </div>

  </div>
</section>

<footer class="bg-gray-800 text-white text-center py-6 mt-10">
  <p>© 2025 جميع الحقوق محفوظة - محرر الصور</p>
</footer>

<script>
  const stripe = Stripe("pk_test_YOUR_PUBLISHABLE_KEY"); // ضع مفتاحك هنا

  document.getElementById("checkout-button").addEventListener("click", async () => {
    const res = await fetch("/api/create-checkout-session", { method: "POST" });
    const session = await res.json();
    const result = await stripe.redirectToCheckout({ sessionId: session.id });
    if(result.error) alert(result.error.message);
  });
</script>

</body>
</html>

<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8">
  <title>تم الدفع بنجاح</title>
  <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
</head>
<body class="bg-green-100 font-sans text-center py-20">
  <div class="max-w-xl mx-auto bg-white p-10 rounded-lg shadow">
    <h1 class="text-3xl font-bold mb-4 text-green-700">🎉 تم الدفع بنجاح!</h1>
    <p class="text-gray-700 mb-6">شكراً لاشتراكك في الخطة الاحترافية. يمكنك الآن استخدام جميع ميزات محرر الصور.</p>
    <a href="index.html" class="bg-blue-600 text-white px-6 py-3 rounded-lg font-bold">العودة للموقع</a>
  </div>
</body>
</html>

<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8">
  <title>تم إلغاء الدفع</title>
  <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
</head>
<body class="bg-red-100 font-sans text-center py-20">
  <div class="max-w-xl mx-auto bg-white p-10 rounded-lg shadow">
    <h1 class="text-3xl font-bold mb-4 text-red-600">❌ تم إلغاء الدفع</h1>
    <p class="text-gray-700 mb-6">لقد تم إلغاء عملية الدفع. يمكنك المحاولة مرة أخرى لاحقاً.</p>
    <a href="index.html" class="bg-blue-600 text-white px-6 py-3 rounded-lg font-bold">العودة للموقع</a>
  </div>
</body>
</html>

import Stripe from "stripe";

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);

export default async function handler(req, res) {
  if(req.method === "POST") {
    try {
      const session = await stripe.checkout.sessions.create({
        payment_method_types: ["card"],
        line_items: [{
          price_data: {
            currency: "usd",
            product_data: { name: "الخطة الاحترافية - محرر الصور" },
            unit_amount: 500,
          },
          quantity: 1
        }],
        mode: "payment",
        success_url: `${req.headers.origin}/success.html`,
        cancel_url: `${req.headers.origin}/cancel.html`
      });
      res.status(200).json({ id: session.id });
    } catch(err) {
      res.status(500).json({ error: err.message });
    }
  } else {
    res.setHeader("Allow", "POST");
    res.status(405).end("Method Not Allowed");
  }
}

{
  "name": "photo-editor-stripe",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "vercel dev"
  },
  "dependencies": {
    "stripe": "^12.0.0"
  }
}
