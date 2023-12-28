資料庫系統期末報告
- Amazon Products -
第四組
M124020013 賴亭涵
M124020017 楊筑鈞
M124020026 簡筠庭
M124020050 陳俞安


**資料集描述**
Amazon Products Dataset 2023資料集
"Amazon Products Dataset 2023"，列出了Amazon上可用的各種產品分類。
該資料集分成兩個資料表：
  1. 資料表一：amazon_cat
    以下為此資料表的欄位：
    ID：每個分類的pk。
    category_name：產品分類的名稱。
  2. 資料表二： amazon_product
	  以下為此資料表的欄位：
    ASIN：亞馬遜標準識別號，用於識別產品的唯一代碼。
    title：產品的名稱或標題。
    imgUrl：產品圖片的網址。
    productURL：產品在亞馬遜頁面的網址。
    stars：產品的平均顧客評分。
    reviews：顧客評論的數量。
    price：產品的當前價格。
    listPrice：產品的標價或建議零售價。
    category_id：參照“亞馬遜分類”資料集中的分類ID。
    isBestSeller：標示該產品是否為暢銷品的boolean。
    boughtInLastMonth：上個月估計購買的單位數。

**迷你世界描述**
  資料表一：AMAZON_CAT
    ID：這是主鍵（PK），每個產品分類都是唯一。
    category_name：這個欄位儲存每個產品分類的名稱，如"Car Care"或"Automotive Replacement Parts"等。
  資料表二：AMAZON_PRIDUCT
    ASIN：每個產品的pk，用於在amazon平台上識別產品。
    title、imgUrl、productURL、stars、reviews、price、listPrice：這些欄位提供了每個產品的詳細資訊，包括名稱、圖片、頁面鏈接、評分、評論數、當前價格和標價。
    category_id：這是外部鍵（FK），它參考amazon_cat
    資料表中的ID。透過category_id，我們可以知道每個產品屬於哪個分類。
    isBestSeller、boughtInLastMonth：提供關於產品市場表現的額外訊息，例如是否為暢銷品和過去一個月的銷售量。
    資料表之間的關聯
    amazon_products資料表中的category_id與amazon_cat資料表中的ID之間存是一對多關係。
    這說明了amazon_cat中的每個分類可以與amazon_products中的多個產品相關聯。例如，"Baby Care Products"分類（在amazon_cat中）可以關聯到多個家電產品（在amazon_products中）。

**查詢句與查詢最佳化策略**
1.查詢句一
查詢從 AMAZON_CAT、amazon_product ap兩張表格的產品分類名稱、產品名稱、評分、評論數、價格，而且這些數據的 boughtInLastMonth >100。

SELECT ac.category_name, ap.title, ap.stars, ap.reviews, ap.price
FROM AMAZON_CAT ac
JOIN amazon_product ap ON ac.ID = ap.category_id
WHERE ap.boughtInLastMonth >100;

優化後:
針對amazon_product中的category_id和boughtInLastMonth建立複合索引INDEX後再進行查詢。

CREATE INDEX idx_category_boughtInLastMonth ON amazon_product(category_id, boughtInLastMonth);
SELECT ac.category_name, ap.title, ap.stars, ap.reviews, ap.price
FROM AMAZON_CAT ac
JOIN amazon_product ap ON ac.ID = ap.category_id
WHERE ap.boughtInLastMonth >100;



2. 查詢句二
從 AMAZON_PRODUCT 資料表選取產品的標題（title）、價格（price）和星級（stars），並通過與 AMAZON_CAT 資料表的連結來獲取每個產品的分類名稱（category_name）。查詢中的 WHERE 子句用於篩選那些星級大於等於 4 且價格低於 100 的產品。

SELECT p.title, p.price, p.stars, c.category_name
FROM AMAZON_PRODUCT p
JOIN AMAZON_CAT c ON p.category_id = c.id
WHERE p.stars >= 4 AND p.price < 100;


優化後:
針對AMAZON_PRODUCT中的category_id和stars，以及AMAZON_CAT中的id建立索引INDEX後再進行查詢。

CREATE INDEX idx_category_id ON AMAZON_PRODUCT(category_id);
CREATE INDEX idx_stars ON AMAZON_PRODUCT(stars);
CREATE INDEX idx_id ON AMAZON_CAT(id);
SELECT p.title, p.price, p.stars, c.category_name
FROM AMAZON_PRODUCT p
JOIN AMAZON_CAT c ON p.category_id = c.id
WHERE p.stars >= 4 AND p.price < 100;

3. 查詢句三
查詢 消費者評價>5 且 星級>3 的產品類別，並按照評論數和價格進行排序，提供一個根據顧客反饋和價格排序的產品列表。

SELECT p.title, p.price, c.category_name
FROM amazon_product p
JOIN (
    SELECT ID, category_name
    FROM amazon_cat
    WHERE ID IN (
        SELECT DISTINCT category_id
        FROM amazon_product
        WHERE stars > 3 AND reviews > 5
    )
) c ON p.category_id = c.ID
ORDER BY p.reviews DESC, p.price ASC;



優化後:
索引對JOIN操作的優化： 針對 amazon_product 的 category_id 與 amazon_cat 的 ID 進行JOIN時，將 amazon_product 表的 category_id 欄位建立索引，可以提升這個JOIN過程的效能。
索引對條件篩選的優化： 針對 stars > 3 AND reviews > 5 ，對 amazon_product 表中的 stars 和 reviews 欄位進行篩選並為這兩個欄位建立複合索引，以提升查詢效能。

CREATE INDEX idx_category_id ON amazon_product(category_id);
CREATE INDEX idx_stars_reviews ON amazon_product(stars, reviews);
SELECT p.title, p.price, c.category_name
FROM amazon_product p
JOIN (
    SELECT ID, category_name
    FROM amazon_cat
    WHERE ID IN (
        SELECT DISTINCT category_id
        FROM amazon_product
        WHERE stars > 3 AND reviews > 5
    )
) c ON p.category_id = c.ID
ORDER BY p.reviews DESC, p.price ASC;
