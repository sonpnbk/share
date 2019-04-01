ALTER PROCEDURE [dbo].[rpt_rev_detail]
    @P_ServiceRateID NVARCHAR(2000) = N'',
    @P_ServiceSubGroupID NVARCHAR(2000) = N'',
    @P_FROMDATE DATETIME = NULL,
    @P_TODATE DATETIME = NULL,
    @P_CASHIER VARCHAR(500) = '',
    @P_SiteCode NVARCHAR(50) = '',
    @P_Channel NVARCHAR(50) = ''
AS
BEGIN
    DECLARE @EmptyGuid UNIQUEIDENTIFIER = '00000000-0000-0000-0000-000000000000';

    SELECT a.InvoiceCode,
           a.OriginInvoiceCode,
           a.AccountCode,
           a.InvoiceDate,
           a.Cashier,
           a.ComputerName,
           a.Quantity,
           a.Price,
           a.VAT,
           (a.TotalMoney - ABS(a.Discount)) AS TotalMoney,
           (a.TotalMoney - a.VAT - ABS(a.Discount)) AS TotalMoneyBeforeTax,
           a.BookingCode,
           a.SessionNo,
           a.ServiceCode,
           a.ZoneName,
		   a.CustomerName,
           a.Description,
           (   SELECT      TOP 1 ssg_i18n.Title
                 FROM      dbo.m_ServicePackage sp
                 LEFT JOIN dbo.m_Service s
                   ON sp.ServiceID               = s.ID
                 LEFT JOIN dbo.m_ServiceSubGroup_i18n ssg_i18n
                   ON ssg_i18n.ServiceSubGroupID = s.ServiceSubGroupID
                WHERE      sp.ID               = a.ServicePackageID
                  AND      LOWER(ssg_i18n.LangCode) = 'vi') AS ServiceSubGroupName,
           (   SELECT      TOP 1 sr_i18n.Title
                 FROM      dbo.m_ServicePackage sp
                 LEFT JOIN dbo.m_ServiceRate_i18n sr_i18n
                   ON sr_i18n.ServiceRateID = sp.ServiceRateID
                WHERE      sp.ID              = a.ServicePackageID
                  AND      LOWER(sr_i18n.LangCode) = 'vi') AS ServiceRateName
      FROM (   SELECT      i.InvoiceCode AS InvoiceCode,
                           oi.InvoiceCode AS OriginInvoiceCode,
                           IIF((a.AccountCode = NULL OR a.AccountCode = ''), a.CardID, a.AccountCode) AS AccountCode,
                           i.InvoiceDate,
                           pl1.FullName AS Cashier,
                           c.Name AS ComputerName,
                           IIF((bd.OriginInvoiceID IS NULL OR bd.OriginInvoiceID = @EmptyGuid), 1, -1) AS 'Quantity',
                           --1 AS 'Quantity',
                           s.ServiceCode,
                           bd.Price,
                           bd.ServicePackageID,
                           bd.Price AS TotalMoney,
                           (   SELECT TOP 1 z_i18n.Title
                                 FROM dbo.m_Zone_i18n z_i18n
                                WHERE LOWER(z_i18n.LangCode) = 'vi'
                                  AND z_i18n.ZoneID          = bd.ZoneID) AS ZoneName,
                           (   SELECT TOP 1 IIF(SUM(bd_vat.Price) IS NULL, 0, SUM(bd_vat.Price))
                                 FROM dbo.BookingDetail bd_vat
                                WHERE bd_vat.IsSplit       = 0
                                  AND bd_vat.RowState      = 2
                                  AND bd_vat.GroupBy       = bd.GroupBy
                                  AND bd_vat.TransactionNo = bd.TransactionNo
                                  AND bd_vat.InvoiceID     = bd.InvoiceID
                           --AND bd_vat.Amount > 0
                           ) AS VAT,
                           (   SELECT TOP 1 IIF(SUM(bd_discount.Price) IS NULL, 0, SUM(bd_discount.Price))
                                 FROM dbo.BookingDetail bd_discount
                                WHERE bd_discount.RowState  = 0
                                  AND bd_discount.GroupBy   = bd.GroupBy
                                  AND bd_discount.InvoiceID = bd.InvoiceID
                                  AND bd_discount.ServiceID = a.ServiceID
                                  AND bd_discount.Price     < 0) AS Discount,
                           b.BookingCode,
                           ss.SessionNo,
                           b.Notes AS Description,
                           (   SELECT TOP 1 c.Name
                                 FROM dbo.BookingCustomer bc
                                 JOIN dbo.Customer c
                                   ON c.ID = bc.CustomerID
                                WHERE bc.BookingID = b.ID) AS CustomerName
                 FROM      dbo.Invoice i
                 JOIN      dbo.BookingDetail bd
                   ON i.ID              = bd.InvoiceID
                 LEFT JOIN dbo.Booking b
                   ON b.ID              = bd.BookingID
                 LEFT JOIN dbo.Invoice oi
                   ON oi.ID             = bd.OriginInvoiceID
                 LEFT JOIN dbo.Account a
                   ON a.BookingDetailID = bd.ID
                 LEFT JOIN dbo.m_PermissionLogin pl
                   ON pl.UserName       = bd.CreatedBy
                 LEFT JOIN dbo.m_Computer c
                   ON c.ID              = bd.ComputerID
                 JOIN      dbo.Session ss
                   ON ss.ID             = bd.SessionID
                 LEFT JOIN dbo.m_PermissionLogin pl1
                   ON pl1.UserName      = ss.Username
                 JOIN      dbo.m_ServicePackage sp
                   ON sp.ID             = bd.ServicePackageID
                 LEFT JOIN dbo.m_Service s
                   ON s.ID              = sp.ServiceID
                WHERE      bd.RowState                          = 0
                  AND      a.Status                                  != 'CANCEL'
                  AND      bd.Price                                  >= 0
                  AND      (   @P_Channel                            = ''
                          OR   @P_Channel LIKE '%' + b.Channel + '%')
                  AND      i.SiteCode                                = @P_SiteCode
                  AND      (   bd.OriginInvoiceID IS NULL
                          OR   bd.OriginInvoiceID                    = @EmptyGuid)
                  AND      DATEDIFF(DAY, @P_FROMDATE, i.InvoiceDate) >= 0
                  AND      DATEDIFF(DAY, i.InvoiceDate, @P_TODATE)   >= 0
                  AND      (   @P_CASHIER                            = ''
                          OR   @P_CASHIER                            = ss.Username)
                  AND      (   @P_ServiceRateID                      = ''
                          OR   @P_ServiceRateID LIKE '%' + CONVERT(NVARCHAR(50), sp.ServiceRateID) + '%')
                  AND      (   @P_ServiceSubGroupID                  = ''
                          OR   @P_ServiceSubGroupID LIKE '%' + CONVERT(NVARCHAR(50), s.ServiceSubGroupID) + '%')) a;
END;

GO

ALTER PROCEDURE [dbo].[rpt_rev_sum]
    @p_ServiceRateID NVARCHAR(500) = '',
    @p_ServiceSubGroupID NVARCHAR(500) = '',
    @P_FROMDATE DATETIME = NULL,
    @P_TODATE DATETIME = NULL,
    @P_CASHIER NVARCHAR(255) = '',
    @P_SITECODE NVARCHAR(255) = '',
	@P_Channel NVARCHAR(50) = ''
AS
BEGIN

	DECLARE @EmptyGuid UNIQUEIDENTIFIER = '00000000-0000-0000-0000-000000000000';

    SELECT SUM(a.TotalMoney) AS TotalMoney,
           --(SUM(a.Quantity)/ SUM(a.UnitQuantity)) AS Quantity,
           (SUM(a.Quantity)) AS Quantity,
           SUM(a.TotalTax) AS TotalTax,
           --(SUM(a.TotalMoney) - SUM(a.TotalTax)) AS TotalMoneyBeforeTax,
           (SUM(a.TotalMoney)*90/100) AS TotalMoneyBeforeTax,
           a.ServiceRateID,
           (
               SELECT TOP 1
                      i18n.Title
               FROM dbo.m_ServiceRate_i18n i18n
               WHERE i18n.ServiceRateID = a.ServiceRateID
                     AND LOWER(i18n.LangCode) = 'vi'
           ) AS ServiceRateName,
           (
               SELECT TOP 1
                      ssg_i18n.Title
               FROM dbo.m_ServicePackage sp
                   LEFT JOIN dbo.m_Service s
                       ON sp.ServiceID = s.ID
                   LEFT JOIN dbo.m_ServiceSubGroup_i18n ssg_i18n
                       ON ssg_i18n.ServiceSubGroupID = s.ServiceSubGroupID
               WHERE sp.ServiceRateID = a.ServiceRateID
                     AND LOWER(ssg_i18n.LangCode) = 'vi'
           ) AS ServiceSubGroupName
    FROM
    (
        SELECT SUM(bd.Amount) AS TotalMoney,
               --SUM(bd.Quantity / sp.UnitQuantity) AS Quantity,
			    --IIF((bd.OriginInvoiceID IS NULL OR bd.OriginInvoiceID = @EmptyGuid), SUM(bd.Quantity / sp.UnitQuantity), SUM(bd.Quantity / sp.UnitQuantity) *-1 ) AS Quantity,
				CASE WHEN ((bd.OriginInvoiceID IS NULL OR bd.OriginInvoiceID = @EmptyGuid) AND bd.PromotionID IS NULL) THEN SUM(bd.Quantity / sp.UnitQuantity)
				   WHEN ((bd.OriginInvoiceID IS NULL OR bd.OriginInvoiceID = @EmptyGuid) AND bd.PromotionID IS NULL ) THEN  SUM(bd.Quantity / sp.UnitQuantity) *-1
				   WHEN (bd.PromotionID != NULL OR bd.PromotionID!= @EmptyGuid) THEN 0
				   END AS Quantity,
               sp.ServiceRateID,
			   SUM(sp.UnitQuantity) AS UnitQuantity,
               (
                   SELECT IIF(SUM(bd_vat.Amount) IS NULL, 0, SUM(bd_vat.Amount))
                   FROM dbo.BookingDetail bd_vat
                   WHERE bd_vat.IsSplit = 0
                         AND bd_vat.RowState = 2
                         AND bd_vat.GroupBy = bd.GroupBy
                         AND bd_vat.BookingID = bd.BookingID
                         AND bd_vat.ServicePackageID = bd.ServicePackageID
						 --AND bd_vat.Amount > 0
               ) AS TotalTax
        FROM dbo.Invoice i
            JOIN dbo.BookingDetail bd
                ON i.ID = bd.InvoiceID
			JOIN dbo.Booking b 
				ON b.ID = bd.BookingID
            JOIN dbo.m_ServicePackage sp
                ON sp.ID = bd.ServicePackageID
            JOIN dbo.Session ss
                ON ss.ID = bd.SessionID
			LEFT JOIN dbo.m_Service s 
				ON s.ID = sp.ServiceID

        WHERE DATEDIFF(DAY, @P_FROMDATE, i.InvoiceDate) >= 0
              AND DATEDIFF(DAY, i.InvoiceDate, @P_TODATE) >= 0
              AND bd.RowState = 0

			  AND (@P_CASHIER = ''  OR ss.Username = @P_CASHIER )
              AND bd.SiteCode = @P_SITECODE
			  AND (@P_Channel = '' OR @P_Channel LIKE '%'+ b.Channel +'%')
			  AND
              (
                  @P_ServiceRateID = ''
                  OR @P_ServiceRateID LIKE '%' + convert(nvarchar(50),sp.ServiceRateID) + '%'
              )
			  AND
              (
                  @P_ServiceSubGroupID = ''
                  OR @P_ServiceSubGroupID LIKE '%' + convert(nvarchar(50),s.ServiceSubGroupID) + '%'
              )
        GROUP BY bd.Quantity,
                 sp.UnitQuantity,
                 sp.ServiceRateID,
				 bd.PromotionID,
                 bd.GroupBy,
                 bd.BookingID,
				 bd.ID,
                 bd.ServicePackageID,
				 bd.OriginInvoiceID
    ) a

    GROUP BY a.ServiceRateID;
END;
GO


ALTER PROCEDURE [dbo].[rpt_rev_by_session]
    @P_FROMDATE DATETIME,
    @P_TODATE DATETIME,
    @P_CASHIER VARCHAR(500),
    @P_SITECODE NVARCHAR(50) = ''
AS
BEGIN

    DECLARE @EmptyGuid UNIQUEIDENTIFIER = '00000000-0000-0000-0000-000000000000';


    SELECT ROW_NUMBER() OVER (PARTITION BY a.SessionNo ORDER BY a.PaymentType) AS SerialNumber,
           CONVERT(NVARCHAR, a.SessionNo) + ' - ' + CONVERT(NVARCHAR, a.StartTime, 103) + ' '
           + LEFT(CONVERT(NVARCHAR, a.StartTime, 108), 5) + ' -> ' + a.EndtimeTemp + ' - ' + a.FullName AS ID,
           a.PaymentType,
           a.SessionNo,
		   SUM(IIF(a.RefundInvoice = 1,a.TotalMoney*-1,a.TotalMoney )) AS TotalMoney,
		   SUM(IIF(a.RefundInvoice = 1,a.TotalVAT*-1,a.TotalVAT )) AS TotalVat,
		 --  (SUM(IIF(a.RefundInvoice = 1,a.TotalMoney*-1,a.TotalMoney )) 
			---  SUM(IIF(a.RefundInvoice = 1,a.TotalVAT*-1,a.TotalVAT ))) AS TotalMoneyBeforeTax,
			 (SUM(IIF(a.RefundInvoice = 1,a.TotalMoney*-1,a.TotalMoney )) * 90/100) AS TotalMoneyBeforeTax
      FROM (   SELECT      CASE
                                WHEN (ss.[Status] = 'SESSION_GOING_ON') THEN 'N/A'
                                ELSE LEFT(CONVERT(NVARCHAR, ss.EndTime, 108), 5)END AS EndtimeTemp,
                           ss.SessionNo,
                           ss.StartTime,
                           ss.ID,
                           c.Name,
                           u.FullName,
                           (   SELECT TOP 1 pt.Title
                                 FROM dbo.m_Service_i18n pt
                                WHERE pt.ServiceID = ipt.PaymentTypeID
                                  AND pt.LangCode  = 'vi') AS PaymentType,
                           IIF(SUM(ipt.Amount) > SUM(i.TotalMoney), SUM(i.TotalMoney), SUM(ipt.Amount)) AS TotalMoney,
                           IIF((   SELECT COUNT(*)
                                     FROM dbo.BookingDetail bd
                                    WHERE bd.InvoiceID           = i.ID
                                      AND (   bd.OriginInvoiceID IS NOT NULL
                                        AND   bd.OriginInvoiceID != @EmptyGuid)) = 0,
                               0,
                               1) AS RefundInvoice,
                           (SELECT COUNT(*) FROM dbo.InvoiceByPaymentType WHERE InvoiceID = i.ID) AS TotalPaymentType,
                           (   SELECT IIF(SUM(bd.Amount) IS NULL, 0, SUM(bd.Amount))
                                 FROM dbo.BookingDetail bd
                                 JOIN dbo.InvoiceByPaymentType ipt1
                                   ON ipt1.InvoiceID = bd.InvoiceID
                                WHERE bd.IsSplit         = 0
                                  AND bd.RowState        = 2
                                  AND bd.SessionID       = ss.ID
                                  AND bd.InvoiceID       = i.ID
                                  AND ipt1.PaymentTypeID = ipt.PaymentTypeID
                                  AND bd.Amount          > 0
								  ) AS TotalVAT
                 FROM      dbo.Session ss
                 JOIN      dbo.Invoice i
                   ON i.SessionID   = ss.ID
                 JOIN      dbo.InvoiceByPaymentType ipt
                   ON ipt.InvoiceID = i.ID
                 LEFT JOIN dbo.m_Computer c
                   ON c.ID          = ss.ComputerID
                 LEFT JOIN dbo.m_PermissionLogin u
                   ON u.UserName    = ss.Username
                WHERE      DATEDIFF(DAY, @P_FROMDATE, ss.StartTime)     >= 0
                  AND      DATEDIFF(DAY, ss.StartTime, @P_TODATE)            >= 0
                  AND      (   @P_SITECODE                                   = ''
                          OR   i.SiteCode                                    = @P_SITECODE)
                  AND      (   @P_CASHIER IS NULL
                          OR   @P_CASHIER                                    = ''
                          OR   @P_CASHIER LIKE '%' + u.UserName + '%')
                GROUP BY ss.SessionNo,
                         ss.ID,
                         ss.StartTime,
                         c.Name,
                         u.FullName,
                         ss.Status,
                         ss.EndTime,
                         ss.Status,
                         ipt.PaymentTypeID,
                         i.ID) a
     GROUP BY a.SessionNo,
              a.PaymentType,
              a.StartTime,
              a.EndtimeTemp,
              a.FullName

     ORDER BY a.SessionNo ASC;
END;
GO
