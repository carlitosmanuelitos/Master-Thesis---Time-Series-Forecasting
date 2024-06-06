SELECT 
    o.code AS 'ORDER_NUMBER',
    o.totalprice AS 'ORDER_AMOUNT_GROSS',
    o.pmiVat AS 'ORDER_VAT',
    o.date AS 'DATE',
    ot.typeCode AS 'ORDER_TYPE',
    CR.couponcode AS 'VOUCHER_ID',
    emp.provisioningexternalid AS 'PLACED_BY',
    oc.code AS 'CHANNEL',
    oe.totalprice AS 'ORDER_ENTRY_AMOUNT_GROSS',
    (oe.baseprice * oe.quantity) - oe.totalprice AS 'ORDER_ENTRY_DISCOUNT_NET',
    oe.pmivatvalue AS 'ORDER_ENTRY_VAT',
    oe.quantity AS 'ENTRY_QUANTITY',
    prd.code AS 'ENTRY_GLOBAL_CODE',
    prd.name AS 'ENTRY_DESCRIPTION',
    sale.code AS 'LOGICAL_CHANNEL',
    rbp.name AS 'PROMOTION_NAME',
    rbp.promotiondescription[en] AS 'PROMOTION_DESCRIPTION',
    'LINE ITEM' AS 'DISCOUNT_TYPE'
FROM Order o
JOIN Orderentry oe ON oe.order = o.pk
LEFT JOIN Product prd ON oe.product = prd.pk
LEFT JOIN OrderType ot ON o.orderType = ot.pk
LEFT JOIN SalesApplication sale ON sale.pk = o.salesApplication
LEFT JOIN OrderChannel oc ON o.orderChannel = oc.pk
LEFT JOIN CouponRedemption CR ON CR.order = o.pk
LEFT JOIN Employee emp ON o.placedby = emp.pk
LEFT JOIN CmsSite site ON o.site = site.pk
LEFT JOIN BaseStore store ON o.store = store.pk AND store.uid LIKE '%-de-%'
JOIN Country country ON store.country = country.pk
JOIN PromotionResult pr ON pr.order = o.pk
JOIN RuleBasedPromotion rbp ON pr.promotion = rbp.pk
JOIN RuleBasedOrderEntryAdjustAction pb ON pb.promotionresult = pr.pk AND pb.orderentryproduct = oe.product AND pb.orderentrynumber = oe.entrynumber AND pb.markedapplied = 1 AND pb.guid = SUBSTRING(oe.discountvaluesinternal, 6, 44)
WHERE o.originalVersion IS NULL
AND o.versionID IS NULL
AND o.status NOT IN ('8796125691995')
AND country.isocode = 'DE'
AND o.creationTime BETWEEN '2024-04-10 00:00:00' AND '2024-04-10 23:59:59'
AND NOT EXISTS (
    SELECT 1
    FROM RuleBasedOrderEntryAdjustAction pb2 
    JOIN PromotionResult pr2 ON pb2.promotionresult = pr2.pk
    JOIN Order o2 ON pr2.order = o2.pk
    JOIN Orderentry oe2 ON oe2.order = o2.pk
    WHERE pb2.promotionresult = pr.pk
    AND pb2.orderentryproduct = oe2.product
    AND pb2.orderentrynumber = oe2.entrynumber
    AND pb2.markedapplied = 1
    AND pb2.guid = SUBSTRING(oe.discountvaluesinternal, 6, 44)
    AND oe2.pk = oe.pk
    AND o2.pk = o.pk
    AND o2.creationTime BETWEEN '2024-04-10 00:00:00' AND '2024-04-10 23:59:59'
)
AND NOT EXISTS (
    SELECT 1
    FROM RuleBasedOrderAdjustTotalAction pb2 
    JOIN PromotionResult pr2 ON pb2.promotionresult = pr2.pk
    JOIN Order o2 ON pr2.order = o2.pk
    WHERE pb2.promotionresult = pr.pk
    AND pb2.markedapplied = 1
    AND pb2.guid = SUBSTRING(oe.discountvaluesinternal, 6, 44)
    AND o2.pk = o.pk
    AND o2.creationTime BETWEEN '2024-04-10 00:00:00' AND '2024-04-10 23:59:59'
);
