const express = require("express");
const axios = require("axios");

const app = express();

app.use(express.json());

const PORT = process.env.PORT || 10000;

app.get("/", (req, res) => {
  res.json({
    app: "Nyumbani",
    status: "running",
  });
});

// M-Pesa access token
async function getAccessToken() {
  const consumerKey = process.env.MPESA_CONSUMER_KEY;
  const consumerSecret = process.env.MPESA_CONSUMER_SECRET;

  const credentials = Buffer.from(
    `${consumerKey}:${consumerSecret}`
  ).toString("base64");

  const response = await axios.get(
    "https://sandbox.safaricom.co.ke/oauth/v1/generate?grant_type=client_credentials",
    {
      headers: {
        Authorization: `Basic ${credentials}`,
      },
    }
  );

  return response.data.access_token;
}

// STK Push
app.post("/api/mpesa/stkpush", async (req, res) => {
  try {
    const { phone, amount } = req.body;

    if (!phone || !amount) {
      return res.status(400).json({
        error: "Phone number and amount are required",
      });
    }

    const accessToken = await getAccessToken();

    const timestamp = new Date()
      .toISOString()
      .replace(/[-:TZ.]/g, "")
      .slice(0, 14);

    const shortCode = process.env.MPESA_SHORTCODE;
    const passkey = process.env.MPESA_PASSKEY;

    const password = Buffer.from(
      `${shortCode}${passkey}${timestamp}`
    ).toString("base64");

    const response = await axios.post(
      "https://sandbox.safaricom.co.ke/mpesa/stkpush/v1/processrequest",
      {
        BusinessShortCode: shortCode,
        Password: password,
        Timestamp: timestamp,
        TransactionType: "CustomerPayBillOnline",
        Amount: Number(amount),
        PartyA: phone,
        PartyB: shortCode,
        PhoneNumber: phone,
        CallBackURL: process.env.MPESA_CALLBACK_URL,
        AccountReference: "Nyumbani",
        TransactionDesc: "Nyumbani booking",
      },
      {
        headers: {
          Authorization: `Bearer ${accessToken}`,
          "Content-Type": "application/json",
        },
      }
    );

    res.json(response.data);
  } catch (error) {
    console.error(error.response?.data || error.message);

    res.status(500).json({
      error: "M-Pesa request failed",
      details: error.response?.data || error.message,
    });
  }
});

// M-Pesa callback
app.post("/api/mpesa/callback", (req, res) => {
  console.log("M-Pesa callback:", JSON.stringify(req.body, null, 2));

  res.json({
    ResultCode: 0,
    ResultDesc: "Accepted",
  });
});

app.listen(PORT, "0.0.0.0", () => {
  console.log(`Nyumbani server running on port ${PORT}`);
});
