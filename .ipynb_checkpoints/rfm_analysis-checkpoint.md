# Portofolio: RFM Analysis - Optimize Business Strategy using Customer Data Analysis

## Introduction

RFM analysis is a method used to analyze customer behaviour using recency, frequency, and monetary where recency answers when is the latest purchase each customer, frequency answers how many times each customer buy products, and monetary answers how much money each customer spend to buy products. 

RFM analysis helps us to increase customer retention, optimize marketing campigns, and identify which customers has high value.

## Dataset and Tools needed

We well use seblak prasmanan database. Seblak prasmanan is one of the new traditional Indonesian food. It's a boiled krupuk with various topping and what make seblak prasmanan special is you can take only topping that you like as many as you want.

This analysis is using POSTGRESQL as the main tool to process the data and the RFM analysis. We will not use all variables to analyze. We only need the id_customer, transaction_date, and money_spent.

## The Methodology

So far, we knew that we need are id_customer, transaction_date, and money_spent. Let's see the metrics we need and how to calculate it:
- To find the recency score, we need to calculate today date minus the newest date the customers bought. For example, customer 03 bought something in '03-04-2023' and today is '06-04-2024'. It means '06-04-2024' - '03-04-2024' = 3 days. The smaller day you get, the better you get score. that's recency.
- To find the frequency score, you just need to count how many times the customers buy. This must be the easiest calculation in RFM analysis.
- To find the monetary is sometimes tricky. Monetary means how much money they spent to buy our products. In this case, we use 'jumlah' variable (which means amount) multiple by 'harga_jual_satuan' (which means price per item).

After that, we will separate them into three categories for each metrics. Recency will get low, medium, and high category. Also the same with Frequency and Monetary. To make it easier to calculate, we will use 1 for low, 2 for medium, and 3 for high.

Finally, we will give weight for each metrics. In this example, we will give weight 20% for recency, 35% for frequency, and 45% for monetary. And then sum them up.

## The Script

```sql
with temp_table as(
select
t.id_pelanggan,
t.tanggal_transaksi,
max(t.tanggal_transaksi) over() as today,
d.jumlah * m.harga_jual_satuan as jumlah
from transaksi t
join detail_transaksi d
on d.id_transaksi = t.id_transaksi
join makanan m
on m.id_makanan = d.id_makanan),
```
'with temp_table as' is to make a temporary table using command table expression technique in sql. Then we select id_pelanggan (which means id_customer), tanggal_transaksi (which means transaction_date).

'max(t.tanggal_transaksi) over() as today' is to show the maximum transaction date and then we write for each row. Also I change the column name into today for convenience sake.

'd.jumlah * m.harga_jual_satuan as jumlah' is to multiple amount and price per each so we get the total money spent.

'from transaksi t join detail_transaksi d on d.id_transaksi = t.id_transaksi join makanan m on m.id_makanan = d.id_makanan)' is needed because the columns we need is from different table so we need to join them first.

```sql
rfm_table as(
select
id_pelanggan,
max(today) - max(tanggal_transaksi) as recency,
count(id_pelanggan) as frequency,
sum(jumlah) as monetary
from temp_table
group by id_pelanggan),
```

'rfm_table as(select' is the second temporary table after we make the first temporary table. Does it mean we make temporary table from temporary table? That's true. The easiest explanation I can give is the first temporary table is the plain table. And this temporary table is a modified table because we do more calculation here. Bear with me.

'select id_pelanggan,' is simply we select id_customer.

'max(today) - max(tanggal_transaksi) as recency,' here we subtract the date of today from the latest transaction date. It will return the different day. Also I change the name into recency.

'count(id_pelanggan) as frequency,' here we count how many times the customers comes to buy again.

'sum(jumlah) as monetary' we sum up the amount of money.

'from temp_table' is we choose the table which is the first temporary table. 

'group by id_pelanggan),' this is important. group by means you group everything by x variable. In this example, we group by id_customer. This will return recency, frequency, and monetary for each id_customer because we GROUP them BY id_customer.

```sql
rfm_score_table as(
select
id_pelanggan,
ntile(3) over(order by recency) as recency_score,
ntile(3) over(order by frequency desc) as frequency_score,
ntile(3) over(order by monetary desc) as monetary_score
from rfm_table)
```

'rfm_score_table as(select' I know... This is the third temporary table. But, doing this can make you easily see the checkpoints. When you make mistake, you know where the mistake is.

'id_pelanggan,' we still want to see the id_customer.

'ntile(3) over(order by X)' means you separate each variable into three groups. You can also make it into five groups by change ntile(3) into ntile(5). The only difference is recency score doesn't use 'descending' because we want the smallest value of recency gets the biggest score of recency.

'from rfm_table)' was the previous temporary table.

```sql
select
id_pelanggan,
(recency_score * 0.2) + 
(frequency_score * 0.35) + 
(monetary_score * 0.45) as rfm_score,
concat(recency_score, frequency_score, monetary_score) as rfm_segment
from rfm_score_table
order by rfm_score desc
```

This is the final step. We select id_customer. We will sum the recency_score multiple by the weight 20%, multiple frequency_score by the weight 35%, and multiple the monetary_score by 45%. So it will loke like: rfm_score = (recency_score * 25%) + (frequency_score * 35%) + (monetary_score * 45%).

we also concat the score for each score to see the segmentation.

## Recommendation Action

Because we know already the segmentation for each customers, so let's count how many customer each segmentation. The syntax would be like this:


Segment 333 has 43 of our most valuable customers, characterized by recency, frequency, and monetary value. These customers shop frequently, spend generously, and have made recent purchases. To retain their loyalty and maximize their potential, consider implementing a retention campaign offering exclusive rewards or benefits. Additionally, upsell or cross-sell premium products, bundles, or complementary items to further increase their spending. Finally, encourage these loyal customers to refer others through attractive referral bonuses to expand your customer base.

Segment 111 consists of 35 customers who are disengaged, exhibiting low recency, frequency, and monetary value. These customers contribute minimally to your revenue. To reactivate them, consider sending personalized emails with enticing offers like discounts, free samples, or a "We Miss You" campaign. Additionally, soliciting feedback can provide valuable insights into the reasons behind their disengagement. Given their low potential, it's advisable to minimize investment in this segment unless feedback suggests significant opportunities for re-engagement.

Segment 222 encompasses 25 customers who exhibit medium levels of recency, frequency, and monetary value. While these mid-tier customers engage moderately, there's potential for growth. To incentivize increased spending, consider offering tiered discounts or loyalty program upgrades. Additionally, engaging these customers with targeted promotions or campaigns tailored to their preferences can foster deeper connections and drive higher customer lifetime value.


















---

### **Segment Insights and Actions**
1. **Segment 333 (High Recency, High Frequency, High Monetary - 43 customers)**  
   - **Insight**: These are your most valuable customers who shop frequently, spend a lot, and have purchased recently.  
   - **Actions**:  
     - **Retention Campaign**: Offer loyalty rewards or exclusive benefits to maintain their engagement.  
     - **Upsell/Cross-sell**: Suggest premium products, bundles, or complementary items.  
     - **Referrals**: Encourage them to refer others through referral bonuses.

2. **Segment 111 (Low Recency, Low Frequency, Low Monetary - 35 customers)**  
   - **Insight**: These customers are disengaged and contribute the least to your revenue.  
   - **Actions**:  
     - **Reactivation Campaign**: Send personalized emails with discounts, free samples, or a "We Miss You" campaign.  
     - **Feedback Request**: Ask for feedback to understand why they stopped engaging.  
     - **Minimal Investment**: Limit spending on this segment unless feedback indicates significant potential.

3. **Segment 222 (Medium Recency, Medium Frequency, Medium Monetary - 25 customers)**  
   - **Insight**: These are mid-tier customers who engage moderately but have room for growth.  
   - **Actions**:  
     - **Incentives to Upgrade**: Offer tiered discounts or loyalty program upgrades to encourage higher spending.  
     - **Engagement**: Share targeted promotions or campaigns tailored to their preferences.  

4. **Segment 211 (High Recency, Low Frequency, Low Monetary - 24 customers)**  
   - **Insight**: Recently engaged but with low frequency and spending.  
   - **Actions**:  
     - **Nurturing Campaign**: Encourage repeat purchases through follow-up reminders or promotions.  
     - **Promotions on Popular Items**: Highlight affordable products or best-sellers to increase their spend.

5. **Segment 122 (Medium Recency, Medium Frequency, Low Monetary - 22 customers)**  
   - **Insight**: They engage somewhat regularly but spend less.  
   - **Actions**:  
     - **Upselling Strategies**: Promote higher-value products.  
     - **Volume Discounts**: Offer incentives for larger purchases.

6. **Segment 322 (High Recency, Medium Frequency, High Monetary - 16 customers)**  
   - **Insight**: These customers recently purchased high-value items and shop moderately.  
   - **Actions**:  
     - **Premium Offers**: Highlight new arrivals or exclusive products.  
     - **Retention**: Send thank-you notes or special rewards to build loyalty.

7. **Segment 233 (Low Recency, High Frequency, High Monetary - 13 customers)**  
   - **Insight**: Previously valuable customers who are now disengaged.  
   - **Actions**:  
     - **Win-Back Campaign**: Offer time-sensitive discounts or personalized messages to re-engage.  
     - **Exclusive Benefits**: Share VIP perks to rekindle their interest.  

8. **Segment 133 (Low Recency, Low Frequency, High Monetary - 7 customers)**  
   - **Insight**: High spenders who are no longer engaged and purchase infrequently.  
   - **Actions**:  
     - **Personalized Outreach**: Follow up with tailored offers based on past high-value purchases.  
     - **Promote Limited-Time Offers**: Create urgency to drive re-engagement.

9. **Segment 311 (High Recency, Low Frequency, Medium Monetary - 5 customers)**  
   - **Insight**: Recent purchasers with moderate spending but low frequency.  
   - **Actions**:  
     - **Post-Purchase Follow-Up**: Offer discounts on their next purchase to encourage repeat visits.  
     - **New Product Recommendations**: Introduce them to other products they may like.

10. **Segment 121 (Medium Recency, Low Frequency, Low Monetary - 1 customer)**  
    - **Insight**: Sporadic engagement with minimal contribution.  
    - **Actions**:  
      - **Awareness Campaign**: Share product highlights or brand stories to build interest.  
      - **Low-Investment Outreach**: Focus on automated marketing for cost-efficiency.

---

### **General Recommendations**
- **Customer Lifecycle Management**:  
   Monitor movement between segments over time. Aim to upgrade customers in lower-value segments to higher-value ones.

- **Data Utilization**:  
   - Use predictive analytics to identify customers at risk of churn.  
   - Analyze what products drive high-value customer behavior (Segment 333 and 322).

- **Marketing Strategy**:  
   - Personalize communication based on RFM segmentation.  
   - Invest more heavily in retaining and growing high-value customers (Segments 333 and 322).

Would you like more detailed recommendations on specific actions or campaigns?




---

Pilihan untuk meningkatkan **Recency**, **Frequency**, atau **Monetary** dalam RFM analisis tergantung pada tujuan bisnis Anda, kondisi pelanggan, dan strategi yang diinginkan. Berikut adalah panduan untuk menentukan prioritas:

---

### **1. Fokus pada Recency**
   **Tujuan:** Mendorong pelanggan untuk kembali melakukan transaksi baru-baru ini.  
   **Kapan Prioritas:** 
   - Ada banyak pelanggan yang sudah lama tidak bertransaksi (skor Recency rendah).
   - Ingin meningkatkan loyalitas pelanggan dan mengurangi churn.
   - Produk Anda bersifat repeatable (bisa dibeli berulang kali, seperti barang konsumsi).  
   **Strategi:**  
   - Kampanye reaktivasi dengan penawaran eksklusif untuk menarik pelanggan yang tidak aktif.
   - Program pengingat otomatis (misalnya, email follow-up setelah periode tertentu tanpa transaksi).  
   **Keuntungan:** 
   - Meningkatkan engagement pelanggan secara langsung.
   - Lebih murah mempertahankan pelanggan lama dibanding mendapatkan pelanggan baru.

---

### **2. Fokus pada Frequency**
   **Tujuan:** Meningkatkan jumlah pembelian per pelanggan.  
   **Kapan Prioritas:**
   - Pelanggan membeli secara sporadis atau jarang (skor Frequency rendah).
   - Ingin memaksimalkan lifetime value (LTV) pelanggan.  
   **Strategi:**  
   - Program loyalitas (reward setelah beberapa pembelian).
   - Bundling produk atau penawaran khusus untuk pembelian berulang.
   - Email retargeting berdasarkan riwayat pembelian.  
   **Keuntungan:**
   - Membantu membangun kebiasaan membeli lebih sering.
   - Meningkatkan pendapatan per pelanggan tanpa harus selalu mencari pelanggan baru.

---

### **3. Fokus pada Monetary**
   **Tujuan:** Meningkatkan nilai pembelian rata-rata per pelanggan.  
   **Kapan Prioritas:**
   - Pelanggan membeli sering, tetapi dengan nilai kecil (skor Monetary rendah).
   - Ingin meningkatkan margin profit dari setiap transaksi.  
   **Strategi:**  
   - Upselling (menawarkan produk lebih premium) dan cross-selling (produk tambahan yang relevan).
   - Tawarkan diskon jika pelanggan mencapai jumlah pembelian tertentu.
   - Perkenalkan produk atau layanan dengan nilai lebih tinggi.  
   **Keuntungan:**
   - Meningkatkan pendapatan dengan pelanggan yang sudah aktif.
   - Mengoptimalkan profitabilitas per transaksi.

---

### **Mana yang Lebih Baik?**
- **Peningkatan Recency:** Cocok jika banyak pelanggan lama yang tidak aktif (menurunkan risiko churn).  
- **Peningkatan Frequency:** Bagus untuk mendorong lebih banyak pembelian dari pelanggan aktif.  
- **Peningkatan Monetary:** Baik jika ingin meningkatkan profitabilitas secara langsung.  

---

### **Saran Praktis: Kombinasi Berimbang**
1. **Identifikasi pelanggan di setiap segmen RFM.** Misalnya, pelanggan dengan Recency rendah tapi Frequency tinggi perlu diaktivasi ulang.
2. **Tentukan prioritas per segmen:** Anda tidak perlu meningkatkan semua dimensi untuk semua pelanggan.
   - Fokus pada **Recency** untuk pelanggan dorman (R1).
   - Fokus pada **Frequency** untuk pelanggan dengan interaksi sporadis (F1/F2).
   - Fokus pada **Monetary** untuk pelanggan aktif yang belanja dalam jumlah kecil (M1/M2).

Dengan pendekatan ini, Anda bisa mengalokasikan sumber daya secara efektif untuk mendapatkan hasil optimal.