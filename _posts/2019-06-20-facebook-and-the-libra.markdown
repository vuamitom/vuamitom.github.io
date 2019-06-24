---
layout: post
title: Why the Libra was created
date: '2019-06-20 00:31:40'
mathjax: false
tags:
- life
---

The day before yesterday, Facebook [annouced its new currency](https://www.facebook.com/zuck/posts/10107693323579671), the Libra as part of its effort to break into the payment market. Trying to eat the payment cake has been a long anticipated move for Facebook, since its Chinese social messaging counter part, WeChat, has demonstrated how widespread mobile payment can be. But unlike, WeChat, Facebook does not only provide a payment service, but it goes as far as to create a blockchain based currency. Why the trouble? 

Imagine that Facebook did it the Whatsapp's way, which is to be a payment provider. It would then sit between customers and merchants in transactions in fiat money (USD, SGD, VND...). In order to buy a cup of tea, the customers would use the Facebook's mobile wallet app to scan the product's QR code, tap agree to pay, and then both the merchant's wallet balance would be credited, and finally the customer can enjoy his cup of tea. But how money goes from the customer's bank account to the merchant's is not that straightforward. Besides flipping bytes and bits to reflect the new balance on the merchant and customer's digital wallet, Facebook would need to rely on existing payment infrastructure to carry out the transaction. They have a few choices. 

The simplest would be to trigger an inter-bank transaction between the user's bank and the merchant's. Interbank payment is a system that enables cross-border transactions between banks which are identifiable by their SWIFT code. If the two banks reside within the same country, they may rely on the national network of payment instead. This approach is naiive as it results in high cost per transaction and it would take up to a few days to complete. Not to mention that if Facebook want to charge a small transaction fee, such micro payment would be prohibitively costly under this system. 

Alternatively, existing payment network such as Visa or Mastercard would happily facilitate those transactions and take a good bite of the cake. This approach may be the simplest but again very costly. 

The above two approaches are either slow or expensive. So one solution that is pretty popular among digital wallets is to make use of the fact that intra-bank transactions are free. Say the merchant and customer to the transaction being discussed hold accounts with bank A and B respectively. Facebook would then open their own accounts with these 2 banks. If a cup of coffee cost 80.000 VND, customer's account would be debitted and Facebook's account at bank B creditted with the same amount. At the same time, for bank A, Facebook's account would be debitted and the merchant's account creditted with 80.000 VND. No money moves out of a bank's balance sheet. I guess this is the approach taken by most local digial wallets like ZaloPay, Momo and ViettelPay. However, Facebook operates at a much larger scale, which results in a more complicated issue. Opening equivalent accounts with major banks in every country in which Facebook has business can be too much of a burden. This is a problem not faced by any other digital wallet providers. Not even WeChat Pay or Alipay. Since the latters mostly serve customers within China. 

So what solution is left for Facebook. Crytocurrency! which allows near instant transaction without middle parties. The reason Facebook wants to issue their own currency instead of adopting existing stable coins (USDT, USDC...) may be technical since none of the existing networks meet the requirement of expected transactions per second. Bitcoin is both volatile in value and vulnerable to high transaction fee (gas price).

At the moment, the move to create the Libra looks like a decision motivated by neccessity for Facebook. Whether they will be able to pursuade law makers, onboard backers and enroll merchants is remained to be seen. If Facebook manages to pull it off, its impact would be far and wide.