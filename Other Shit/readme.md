SELECT
       x.ORDER_NUMBER,
       x.CHANNEL,
       x.ORDER_TYPE,
       x.LOGICAL_CHANNEL,
       x.DATE,
       x.ORDER_ENTRY_AMOUNT_GROSS,
       x.ORDER_ENTRY_VAT,
       x.ORDER_ENTRY_DISCOUNT_NET,
       x.VOUCHER_ID,
       x.PLACED_BY,
       x.ENTRY_GLOBAL_CODE,
       x.ENTRY_DESCRIPTION,
       x.ENTRY_QUANTITY,
       x.PROMOTION_NAME,
       x.PROMOTION_DESCRIPTION,
       x.DISCOUNT_TYPE,
       x.ORDER_AMOUNT_GROSS,
       x.ORDER_VAT
FROM (
    {{
    SELECT DISTINCT
          {o:code} AS 'ORDER_NUMBER',
          {ot:typeCode} AS 'ORDER_TYPE',
          {oc:code} AS 'CHANNEL',
          {sale:code} AS 'LOGICAL_CHANNEL',
          {o:totalprice} AS 'ORDER_AMOUNT_GROSS',
          {o:pmiVat} AS 'ORDER_VAT',
          {o:date} AS 'DATE',
          {CR.couponcode} AS 'VOUCHER_ID',
          {emp:provisioningexternalid} AS 'PLACED_BY',
          {oe:totalprice} AS 'ORDER_ENTRY_AMOUNT_GROSS',
          ({oe:baseprice} * ({oe:quantity})) - {oe:totalprice} AS 'ORDER_ENTRY_DISCOUNT_NET',
          {oe:pmivatvalue} AS 'ORDER_ENTRY_VAT',
          {oe:quantity} AS 'ENTRY_QUANTITY',
          {prd:code} AS 'ENTRY_GLOBAL_CODE',
          {prd:name} AS 'ENTRY_DESCRIPTION',
          {rbp:name} AS 'PROMOTION_NAME',
          {rbp:promotiondescription[en]} AS 'PROMOTION_DESCRIPTION',
          'LINE ITEM' AS 'DISCOUNT_TYPE'
    FROM {Order AS o
        JOIN Orderentry AS oe ON {oe:order} = {o:pk}
        LEFT JOIN OrderType AS ot ON {o:orderType}={ot:pk}
        LEFT JOIN SalesApplication AS sale ON {sale:pk}={o:salesApplication}
        LEFT JOIN OrderChannel AS oc ON {o:orderChannel} = {oc:pk}
        LEFT JOIN Product AS prd ON {oe:product} = {prd:pk}
        LEFT JOIN CouponRedemption AS CR ON {CR.order} = {o.pk}
        LEFT JOIN Employee AS emp ON {o:placedby} = {emp:pk}
        LEFT JOIN CmsSite AS site ON {o:site} = {site:pk}
        LEFT JOIN BaseStore AS store ON {o:store} = {store:pk} AND {store:uid} LIKE '%-de-%'
        JOIN Country AS country ON {store:country} = {country:pk}
        JOIN PromotionResult AS pr ON {pr:order} = {o:pk}
        JOIN RuleBasedPromotion AS rbp ON {pr:promotion} = {rbp:pk}
        JOIN RuleBasedOrderEntryAdjustAction AS pb ON {pb:promotionresult} = {pr:pk}
            AND {pb:orderentryproduct} = {oe:product}
            AND {pb:orderentrynumber} = {oe:entrynumber}
            AND {pb:markedapplied} = 1
            AND {pb:guid} = SUBSTRING({oe.discountvaluesinternal}, 6, 44)
    }
    WHERE {o:originalVersion} IS NULL
    AND {o:versionID} IS NULL
    AND {o:status} NOT IN ('8796125691995')
    AND {country.isocode} = 'DE'
    AND {o:creationTime} BETWEEN CAST('2024-04-01 00:00:00' AS DATETIME) AND CAST('2024-04-01 00:15:00' AS DATETIME)
    }}
    UNION ALL
    {{
    SELECT DISTINCT
          {o:code} AS 'ORDER_NUMBER',
          {ot:typeCode} AS 'ORDER_TYPE',
          {oc:code} AS 'CHANNEL',
          {sale:code} AS 'LOGICAL_CHANNEL',
          {o:totalprice} AS 'ORDER_AMOUNT_GROSS',
          {o:pmiVat} AS 'ORDER_VAT',
          {o:date} AS 'DATE',
          {CR.couponcode} AS 'VOUCHER_ID',
          {emp:provisioningexternalid} AS 'PLACED_BY',
          {oe:totalprice} AS 'ORDER_ENTRY_AMOUNT_GROSS',
          ({oe:baseprice} * ({oe:quantity})) - {oe:totalprice} AS 'ORDER_ENTRY_DISCOUNT_NET',
          {oe:pmivatvalue} AS 'ORDER_ENTRY_VAT',
          {oe:quantity} AS 'ENTRY_QUANTITY',
          {prd:code} AS 'ENTRY_GLOBAL_CODE',
          {prd:name} AS 'ENTRY_DESCRIPTION',
          {rbp:name} AS 'PROMOTION_NAME',
          {rbp:promotiondescription[en]} AS 'PROMOTION_DESCRIPTION',
          'LINE ITEM' AS 'DISCOUNT_TYPE'
    FROM {Order AS o
        JOIN Orderentry AS oe ON {oe:order} = {o:pk}
        LEFT JOIN OrderType AS ot ON {o:orderType}={ot:pk}
        LEFT JOIN SalesApplication AS sale ON {sale:pk}={o:salesApplication}
        LEFT JOIN OrderChannel AS oc ON {o:orderChannel} = {oc:pk}
        LEFT JOIN Product AS prd ON {oe:product} = {prd:pk}
        LEFT JOIN OrderStatus AS hst ON {o:status} = {hst:pk}
        LEFT JOIN CouponRedemption AS CR ON {CR.order} = {o.pk}
        LEFT JOIN Employee AS emp ON {o:placedby} = {emp:pk}
        LEFT JOIN CmsSite AS site ON {o:site} = {site:pk}
        LEFT JOIN BaseStore AS store ON {o:store} = {store:pk} AND {store:uid} LIKE '%-de-%'
        JOIN Country AS country ON {store:country} = {country:pk}
        JOIN PromotionResult AS pr ON {pr:order} = {o:pk}
        JOIN RuleBasedPromotion AS rbp ON {pr:promotion} = {rbp:pk}
    }
    WHERE {o:originalVersion} IS NULL
    AND {o:versionID} IS NULL
    AND {o:status} NOT IN ('8796125691995')
    AND {country.isocode} = 'DE'
    AND {o:creationTime} BETWEEN CAST('2024-04-01 00:00:00' AS DATETIME) AND CAST('2024-04-01 00:15:00' AS DATETIME)
    AND NOT EXISTS (
        {{
            SELECT 1
            FROM {RuleBasedOrderEntryAdjustAction AS pb2
                  JOIN PromotionResult AS pr ON {pb2:promotionresult} = {pr:pk}
                  JOIN Order AS o2 ON {pr:order} = {o2:pk}
                  JOIN Orderentry AS oe2 ON {oe2:order} = {o2:pk}
            }
            WHERE {pb2:promotionresult} = {pr:pk}
              AND {pb2:orderentryproduct} = {oe2:product}
              AND {pb2:orderentrynumber} = {oe2:entrynumber}
              AND {pb2:markedapplied} = 1
              AND {pb2:guid} = SUBSTRING({oe.discountvaluesinternal}, 6, 44)
              AND {oe2:pk} = {oe:pk}
              AND {o2:pk} = {o:pk}
              AND {o2:creationTime} BETWEEN CAST('2024-04-01 00:00:00' AS DATETIME) AND CAST('2024-04-01 00:15:00' AS DATETIME)
        }}
    )
    AND NOT EXISTS (
        {{
            SELECT 1
            FROM {RuleBasedOrderAdjustTotalAction AS pb2
                  JOIN PromotionResult AS pr ON {pb2:promotionresult} = {pr:pk}
                  JOIN Order AS o2 ON {pr:order} = {o2:pk}
            }
            WHERE {pb2:promotionresult} = {pr:pk}
              AND {pb2:markedapplied} = 1
              AND {pb2:guid} = SUBSTRING({oe.discountvaluesinternal}, 6, 44)
              AND {o2:pk} = {o:pk}
              AND {o2:creationTime} BETWEEN CAST('2024-04-01 00:00:00' AS DATETIME) AND CAST('2024-04-01 00:15:00' AS DATETIME)
        }}
    )
    }}
) x
