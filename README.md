const express = require('express');
const Stripe = require('stripe');
const cors = require('cors');

env.config();
const stripe = Stripe(process.env.STRIPE_SECRET_KEY);

const app = express();
app.use(express.json());
app.use(cors());

app.post('/create-checkout-session', async (req, res) => {
    try {
        const session = await stripe.checkout.sessions.create({
            payment_method_types: ['card'],
            line_items: [
                {
                    price_data: {
                        currency: 'ron',
                        product_data: {
                            name: 'Produse HappBar',
                        },
                        unit_amount: req.body.amount * 100,
                    },
                    quantity: 1,
                },
            ],
            mode: 'payment',
            success_url: 'http://localhost:3000/success',
            cancel_url: 'http://localhost:3000/cancel',
        });

        res.json({ url: session.url });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

app.get('/success', (req, res) => {
    res.send(`
        <html>
        <head>
            <style>
                body { text-align: center; padding: 50px; font-family: Arial, sans-serif; }
                h1 { color: green; }
                a { text-decoration: none; color: white; background: green; padding: 10px 20px; border-radius: 5px; }
            </style>
        </head>
        <body>
            <h1>Plata a fost efectuată cu succes! 🎉</h1>
            <p>Mulțumim pentru achiziție. Vei primi detaliile comenzii pe email.</p>
            <a href="/">Înapoi la magazin</a>
        </body>
        </html>
    `);
});

app.get('/cancel', (req, res) => {
    res.send(`
        <html>
        <head>
            <style>
                body { text-align: center; padding: 50px; font-family: Arial, sans-serif; }
                h1 { color: red; }
                a { text-decoration: none; color: white; background: red; padding: 10px 20px; border-radius: 5px; }
            </style>
        </head>
        <body>
            <h1>Plata a fost anulată ❌</h1>
            <p>Se pare că ai anulat procesul de plată. Încercați din nou.</p>
            <a href="/">Înapoi la magazin</a>
        </body>
        </html>
    `);
});

app.listen(5000, () => console.log('Server running on port 5000'));
