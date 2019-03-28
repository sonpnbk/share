USE [TicketTTC_TLTY]
GO
/****** Object:  StoredProcedure [dbo].[Rpt_Booking_Detail_ByProfile]    Script Date: 3/28/2019 1:54:05 PM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO


-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date,,>
-- Description:	<Description,,>
-- =============================================
CREATE PROCEDURE [dbo].[Rpt_Booking_Detail_ByProfile]
    @ProfileCode NVARCHAR(50) = '',
    @ProfileName NVARCHAR(255) = '',
    @TaxNo VARCHAR(20) = '',
    @bookingStatus VARCHAR(255) = 'CONFIRM',
    @startDate DATETIME = NULL,
    @endDate DATETIME = NULL,
    @siteId UNIQUEIDENTIFIER = ''
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE @defaultProfileCodeB2B VARCHAR(5) = 'TA';
    DECLARE @defaultProfileCodeB2C VARCHAR(20) = 'TRAVELLER';

    SELECT p.ProfileCode,
           p.Name AS ProfileName,
           b.CheckinDate,
           b.BookingCode,
           b.CreatedBy,
           b.CreatedDate,
           b.TotalAmount,
		   (SELECT TOP 1 SUM(a.TotalQuantity) FROM  (SELECT  
						sp.ServiceRateID,
						(bd.Quantity/sp.UnitQuantity ) AS TotalQuantity
						FROM dbo.BookingDetail bd 
						JOIN dbo.m_ServicePackage sp ON bd.ServicePackageID = sp.ID
						WHERE bd.RowState = 0 AND bd.BookingID = b.Id
		   ) a GROUP BY a.ServiceRateID,a.TotalQuantity) AS TotalQuantity,
		   dbo.fnISOweekOfMonth(b.CheckinDate) AS [Week],
		   0 AS TotalDiscount,
		   0 AS TotalDiscountPercent,
           (
               SELECT TOP 1
                      c.Name
               FROM dbo.BookingCustomer bc
                   JOIN dbo.Customer c
                       ON c.ID = bc.CustomerID
                          AND bc.BookingID = b.ID
                          AND c.CustomerType = 'CUSTOMER'
           ) AS CustomerName,
           (
               SELECT TOP 1
                      c.Email
               FROM dbo.BookingCustomer bc
                   JOIN dbo.Customer c
                       ON c.ID = bc.CustomerID
                          AND bc.BookingID = b.ID
                          AND c.CustomerType = 'CUSTOMER'
           ) AS CustomerEmail,
           (
               SELECT TOP 1
                      c.PhoneNumber
               FROM dbo.BookingCustomer bc
                   JOIN dbo.Customer c
                       ON c.ID = bc.CustomerID
                          AND bc.BookingID = b.ID
                          AND c.CustomerType = 'CUSTOMER'
           ) AS CustomerPhonenumber
    FROM dbo.Booking b
        JOIN dbo.m_Profile p
            ON b.ProfileID = p.ID
    WHERE b.ProfileID IN
          (
              SELECT ID
              FROM dbo.m_Profile
              WHERE ProfileType IN ( @defaultProfileCodeB2B, @defaultProfileCodeB2C )
          )
          AND b.BookingStatus LIKE '%' + @bookingStatus + '%'
          AND DATEDIFF(DAY, @startDate, b.CheckinDate) >= 0
          AND DATEDIFF(DAY, b.CheckinDate, @endDate) >= 0
          AND
          (
              @TaxNo = ''
              OR p.IdentityCard LIKE '%' + @TaxNo + '%'
          )
          AND
          (
              @ProfileCode = ''
              OR p.ProfileCode LIKE '%' + @ProfileCode + '%'
          )
          AND
          (
              @ProfileName = ''
              OR p.Name LIKE '%' + @ProfileName + '%'
          )
          AND b.SiteID = @siteId
		  AND b.BookingStatus LIKE '%'+@bookingStatus+'%'
		  ORDER BY b.CreatedDate ASC;
END;
GO
/****** Object:  StoredProcedure [dbo].[Rpt_Booking_Summary_Revenue_B2B]    Script Date: 3/28/2019 1:54:05 PM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO


-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date,,>
-- Description:	<Description,,>
-- =============================================
CREATE PROCEDURE [dbo].[Rpt_Booking_Summary_Revenue_B2B]
	@ProfileCode NVARCHAR(50) = '' ,
    @ProfileName NVARCHAR(255) = '' ,
	@TaxNo VARCHAR(20) = '' ,
    @bookingStatus VARCHAR(255) = 'CONFIRM',
    @startDate DATETIME = NULL,
    @endDate DATETIME = NULL,
	@siteId UNIQUEIDENTIFIER = ''
AS
BEGIN
    SET NOCOUNT ON;

	DECLARE @defaultProfileCodeB2B VARCHAR(5) = 'TA';
	DECLARE @defaultProfileCodeB2C VARCHAR(20) = 'TRAVELLER';
    SELECT p.IdentityCard AS TaxNo,
           p.ProfileCode,
           p.Name AS ProfileName,
           p.Address,
           p.ProfileType,
           SUM(b.TotalAmount) AS TotalMoney
    FROM dbo.Booking b
        JOIN dbo.m_Profile p
            ON p.ID = b.ProfileID
    WHERE b.ProfileID IN
          (
              SELECT ID FROM dbo.m_Profile WHERE ProfileType IN (@defaultProfileCodeB2B,@defaultProfileCodeB2C)
          )
          AND b.BookingStatus LIKE '%' + @bookingStatus + '%'
		  AND DATEDIFF(DAY, @startDate, b.CheckinDate) >= 0
		  AND DATEDIFF(DAY, b.CheckinDate, @endDate) >= 0
		  AND (@TaxNo = '' OR p.IdentityCard LIKE '%'+ @TaxNo +'%')
		  AND (@ProfileCode = '' OR p.ProfileCode LIKE '%'+ @ProfileCode +'%')
		  AND (@ProfileName = '' OR p.Name LIKE '%'+ @ProfileName +'%')
		  AND b.SiteID = @siteId
    GROUP BY b.ProfileID,
             p.IdentityCard,
             p.ProfileCode,
             p.Address,
             p.ProfileType,
             p.Name;


END;
GO
/****** Object:  StoredProcedure [dbo].[Rpt_DetailsService]    Script Date: 3/28/2019 1:54:05 PM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date,,>
-- Description:	<Description,,>
-- =============================================
CREATE PROCEDURE [dbo].[Rpt_DetailsService]
    @fromDate DATETIME = NULL,
    @toDate DATETIME = NULL,
    @ServiceGroupID VARCHAR(2000) = '',
    @ServiceID VARCHAR(2000) = '',
    @Status VARCHAR(200) = '',
    @siteCode NVARCHAR(50) = ''
AS
BEGIN
    SET NOCOUNT ON;


    IF (@Status = 'USED')
    BEGIN
        SELECT i.InvoiceCode,
               IIF(a.AccountCode = '' OR a.AccountCode IS NULL, a.CardID, a.AccountCode) AS ServiceCode,
               bd.SaleDate,
               a.IssuedDate,
               a.ExpirationDate,
               (   SELECT TOP 1 s_i18n.Title
                     FROM dbo.m_Service_i18n s_i18n
                    WHERE s_i18n.ServiceID       = bd.ServiceID
                      AND LOWER(s_i18n.LangCode) = 'vi') AS ServiceName,
               (   SELECT      TOP 1 ssg_i18n.Title
                     FROM      dbo.m_ServicePackage sp
                     LEFT JOIN dbo.m_Service s
                       ON sp.ServiceID               = s.ID
                     LEFT JOIN dbo.m_ServiceSubGroup_i18n ssg_i18n
                       ON ssg_i18n.ServiceSubGroupID = s.ServiceSubGroupID
                    WHERE      sp.ID               = bd.ServicePackageID
                      AND      LOWER(ssg_i18n.LangCode) = 'vi') AS ServiceGroup,
               1 AS Quantity,
               bd.Price,
               a.Status,
               N'Đã sử dụng' AS 'StatusStr',
               (   SELECT IIF(SUM(bd_discount.Amount) IS NULL, 0, SUM(bd_discount.Amount))
                     FROM dbo.BookingDetail bd_discount
                    WHERE bd_discount.IsSplit          = 0
                      AND bd_discount.RowState         = 0
                      AND bd_discount.GroupBy          = bd.GroupBy
                      AND bd_discount.BookingID        = bd.BookingID
                      AND bd_discount.ServiceID        = bd.ServiceID
                      AND bd_discount.ServicePackageID = bd.ServicePackageID
                      AND bd_discount.Amount           < 0) AS TotalDiscount
          FROM dbo.Invoice i
          JOIN dbo.BookingDetail bd
            ON i.ID          = bd.InvoiceID
          JOIN dbo.Account a
            ON bd.ID         = a.BookingDetailID
          JOIN dbo.AccountUsingDetail aud
            ON aud.AccountID = a.ID
          JOIN dbo.m_Service s
            ON s.ID          = a.ServiceID
          JOIN dbo.m_ServicePackage sp
            ON sp.ID         = bd.ServicePackageID
         WHERE i.Inactive                            = 0
           AND s.Inactive                            = 0
           AND bd.RowState                           = 0
           AND bd.Price                              > 0
           AND DATEDIFF(DAY, bd.SaleDate, @fromDate) <= 0
           AND DATEDIFF(DAY, bd.SaleDate, @toDate)   >= 0
           AND i.SiteCode                            = @siteCode
           AND (   @ServiceID                        = ''
              OR   @ServiceID LIKE '%' + CONVERT(NVARCHAR(50), sp.ServiceRateID) + '%')
           AND (   @ServiceGroupID                   = ''
              OR   @ServiceGroupID LIKE '%' + CONVERT(NVARCHAR(50), s.ServiceSubGroupID) + '%');
    END;
    ELSE
    BEGIN

        SELECT i.InvoiceCode,
               IIF(a.AccountCode = '' OR a.AccountCode IS NULL, a.CardID, a.AccountCode) AS ServiceCode,
               bd.SaleDate,
               a.IssuedDate,
               a.ExpirationDate,
               (   SELECT TOP 1 s_i18n.Title
                     FROM dbo.m_Service_i18n s_i18n
                    WHERE s_i18n.ServiceID       = bd.ServiceID
                      AND LOWER(s_i18n.LangCode) = 'vi') AS ServiceName,
               (   SELECT      TOP 1 ssg_i18n.Title
                     FROM      dbo.m_ServicePackage sp
                     LEFT JOIN dbo.m_Service s
                       ON sp.ServiceID               = s.ID
                     LEFT JOIN dbo.m_ServiceSubGroup_i18n ssg_i18n
                       ON ssg_i18n.ServiceSubGroupID = s.ServiceSubGroupID
                    WHERE      sp.ID               = bd.ServicePackageID
                      AND      LOWER(ssg_i18n.LangCode) = 'vi') AS ServiceGroup,
               1 AS Quantity,
               bd.Price,
               a.Status,
               CASE a.Status
                    WHEN 'INACTIVE' THEN N'Chờ kích hoạt'
                    WHEN 'ACTIVE' THEN N'Chờ sử dụng'
                    WHEN 'LOCK' THEN N'Khóa'
                    WHEN 'CLOSE' THEN N'Ngừng sử dụng'
                    WHEN 'CANCEL' THEN N'Hủy' END AS StatusStr,
               (   SELECT IIF(SUM(bd_discount.Amount) IS NULL, 0, SUM(bd_discount.Amount))
                     FROM dbo.BookingDetail bd_discount
                    WHERE bd_discount.IsSplit          = 0
                      AND bd_discount.RowState         = 0
                      AND bd_discount.GroupBy          = bd.GroupBy
                      AND bd_discount.BookingID        = bd.BookingID
                      AND bd_discount.ServiceID        = bd.ServiceID
                      AND bd_discount.ServicePackageID = bd.ServicePackageID
                      AND bd_discount.Amount           < 0) AS TotalDiscount
          FROM dbo.Invoice i
          JOIN dbo.BookingDetail bd
            ON i.ID  = bd.InvoiceID
          JOIN dbo.Account a
            ON bd.ID = a.BookingDetailID
          JOIN dbo.m_Service s
            ON s.ID  = a.ServiceID
          JOIN dbo.m_ServicePackage sp
            ON sp.ID = bd.ServicePackageID
         WHERE i.Inactive                            = 0
           AND s.Inactive                            = 0
           AND a.Status                              = @Status
           AND DATEDIFF(DAY, bd.SaleDate, @fromDate) <= 0
           AND DATEDIFF(DAY, bd.SaleDate, @toDate)   >= 0
           AND i.SiteCode                            = @siteCode
           AND bd.RowState                           = 0
           AND bd.Price                              > 0
           AND (   SELECT COUNT(*)
                     FROM dbo.AccountUsingDetail aud
                    WHERE aud.AccountID = a.ID)      = 0
           AND (   @ServiceID                        = ''
              OR   @ServiceID LIKE '%' + CONVERT(NVARCHAR(50), sp.ServiceRateID) + '%')
           AND (   @ServiceGroupID                   = ''
              OR   @ServiceGroupID LIKE '%' + CONVERT(NVARCHAR(50), s.ServiceSubGroupID) + '%');

    END;



END;
GO
/****** Object:  StoredProcedure [dbo].[Rpt_DetailUseServiceByACM]    Script Date: 3/28/2019 1:54:05 PM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO


-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date,,>
-- Description:	<Description,,>
-- =============================================
CREATE PROCEDURE [dbo].[Rpt_DetailUseServiceByACM]
    @startDate DATETIME = NULL,
    @endDate DATETIME = NULL,
    @siteCode NVARCHAR(50) = ''
AS
BEGIN
    SET NOCOUNT ON;

    SELECT asd.UsingDate,
           IIF((a.AccountCode IS NULL OR a.AccountCode = ''), a.CardID, a.AccountCode) AS AccountCode,
           c.Name AS ACM,
           CASE
               WHEN s.CardType = 'QRCODE' THEN
                   'QrCode'
               WHEN s.CardType = 'CARD' THEN
                   'Thẻ'
               ELSE
                   ''
           END AS CardType,
           (
               SELECT TOP 1
                      sg_i18n.Title
               FROM dbo.m_ServiceSubGroup_i18n sg_i18n
               WHERE sg_i18n.ServiceSubGroupID = s.ServiceSubGroupID
                     AND LOWER(sg_i18n.LangCode) = 'vi'
           ) AS ServiceSubGroupName,
           pl.FullName AS Cashier,
           i.InvoiceCode,
           (
               SELECT TOP 1
                      sr_i18n.Title
               FROM dbo.m_ServicePackage sp
                   LEFT JOIN dbo.m_ServiceRate_i18n sr_i18n
                       ON sr_i18n.ServiceRateID = sp.ServiceRateID
               WHERE sp.ID = bd.ServicePackageID
                     AND LOWER(sr_i18n.LangCode) = 'vi'
           ) AS ServiceRateName
    FROM dbo.AccountUsingDetail asd
        JOIN dbo.m_Computer c
            ON asd.ComputerID = c.ID
        JOIN dbo.Account a
            ON asd.AccountID = a.ID
        JOIN dbo.m_Service s
            ON a.ServiceID = s.ID
        LEFT JOIN dbo.m_PermissionLogin pl
            ON pl.UserName = a.CreatedBy
        LEFT JOIN dbo.BookingDetail bd
            ON bd.ID = a.BookingDetailID
        LEFT JOIN dbo.Invoice i
            ON i.ID = bd.InvoiceID
	WHERE DATEDIFF(DAY, @startDate, asd.UsingDate) >= 0
    AND DATEDIFF(DAY, asd.UsingDate, @endDate) >= 0
	AND asd.SiteCode = @siteCode;



END;
GO
/****** Object:  StoredProcedure [dbo].[Rpt_RefundService]    Script Date: 3/28/2019 1:54:05 PM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date,,>
-- Description:	<Description,,>
-- =============================================
CREATE PROCEDURE [dbo].[Rpt_RefundService]
    -- Add the parameters for the stored procedure here
    @fromDate DATETIME = '2018/1/20',
    @toDate DATETIME = '2018/1/20',
    @cashierID NVARCHAR(500) = ''
AS
BEGIN
    SET NOCOUNT ON;

    SELECT      ROW_NUMBER() OVER (PARTITION BY oi.CreatedBy ORDER BY bd.SaleDate) AS [SerialNumber],
                bd.SaleDate,
                a.CreatedBy,
				pl.FullName AS [Cashier],
                CASE
                     WHEN a.CardID <> '' THEN a.CardID
                     ELSE a.AccountCode END AS [ServiceCode],
                (   SELECT TOP 1 ssg_i18n.Title
                      FROM dbo.m_ServiceSubGroup_i18n ssg_i18n
                     WHERE LOWER(ssg_i18n.LangCode)   = 'vi'
                       AND ssg_i18n.ServiceSubGroupID = s.ServiceSubGroupID) AS ServiceSubGroupName,
                (
				SELECT TOP 1 Title FROM dbo.m_Service_i18n s_i18n WHERE
					s_i18n.ServiceID = bd.ServiceID AND LOWER(s_i18n.LangCode) = 'vi'
				) AS [ServiceName],
                bd.Price AS TotalMoney,
                i.Description,
                i.InvoiceCode,
                oi.InvoiceCode AS OriginInvoiceCode
      FROM      dbo.BookingDetail bd WITH (NOLOCK)
      JOIN      dbo.[m_Service] s WITH (NOLOCK)
        ON s.ID               = bd.ServiceID
      JOIN
                dbo.Invoice i WITH (NOLOCK)
        ON bd.InvoiceID       = i.ID
      JOIN      dbo.Invoice oi WITH (NOLOCK)
        ON bd.OriginInvoiceID = oi.ID
      LEFT JOIN dbo.Account a WITH (NOLOCK)
        ON a.BookingDetailID  = bd.ID
	  LEFT JOIN dbo.m_PermissionLogin pl WITH (NOLOCK)
		ON pl.UserName = oi.CreatedBy
     WHERE      bd.Amount                                < 0
       AND      bd.RowState                              = 0
       AND      DATEDIFF(DAY, oi.InvoiceDate, @FromDate) <= 0
       AND      DATEDIFF(DAY, oi.InvoiceDate, @ToDate)   >= 0
	   AND (@CashierID = '' OR (oi.CreatedBy) = @cashierID);

END;
GO
/****** Object:  StoredProcedure [dbo].[rpt_rev_by_session]    Script Date: 3/28/2019 1:54:05 PM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO


CREATE PROCEDURE [dbo].[rpt_rev_by_session]
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
/****** Object:  StoredProcedure [dbo].[rpt_rev_detail]    Script Date: 3/28/2019 1:54:05 PM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO




CREATE PROCEDURE [dbo].[rpt_rev_detail]
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
/****** Object:  StoredProcedure [dbo].[rpt_rev_sum]    Script Date: 3/28/2019 1:54:05 PM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO


CREATE PROCEDURE [dbo].[rpt_rev_sum]
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
/****** Object:  StoredProcedure [dbo].[Rpt_ServiceSales]    Script Date: 3/28/2019 1:54:05 PM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date,,>
-- Description:	<Description,,>
-- =============================================
CREATE PROCEDURE [dbo].[Rpt_ServiceSales] @sessionId UNIQUEIDENTIFIER = NULL
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE @EmptyGuid UNIQUEIDENTIFIER = '00000000-0000-0000-0000-000000000000';


    SELECT a.ServiceRateName,
           SUM(a.Quantity) AS Quantity,
           a.Price,
           SUM(a.TotalTax) AS TotalTax,
           SUM(a.Amount - a.TotalDiscount) AS TotalMoney,
           SUM(a.TotalDiscount) AS TotalDiscount
      FROM (   SELECT (   SELECT TOP 1 i18n.Title
                            FROM dbo.m_ServiceRate_i18n i18n
                           WHERE i18n.ServiceRateID   = sp.ServiceRateID
                             AND LOWER(i18n.LangCode) = 'vi') AS ServiceRateName,
                      (bd.Quantity / sp.UnitQuantity) AS Quantity,
                      bd.Price,
                      sp.ServiceRateID,
                      i.CreatedDate,
                      (   SELECT IIF(SUM(bd_vat.Amount) IS NULL, 0, SUM(bd_vat.Amount))
                            FROM dbo.BookingDetail bd_vat
                           WHERE bd_vat.IsSplit          = 0
                             AND bd_vat.RowState         = 2
                             AND bd_vat.GroupBy          = bd.GroupBy
                             AND bd_vat.BookingID        = bd.BookingID
                             AND bd_vat.ServicePackageID = bd.ServicePackageID
                             AND bd_vat.Amount           > 0) AS TotalTax,
                      (bd.Amount) AS Amount,
                      (   SELECT IIF(SUM(bd_vat.Amount) IS NULL, 0, SUM(bd_vat.Amount))
                            FROM dbo.BookingDetail bd_vat
                           WHERE bd_vat.IsSplit          = 0
                             AND bd_vat.RowState         = 0
                             AND bd_vat.GroupBy          = bd.GroupBy
                             AND bd_vat.BookingID        = bd.BookingID
                             AND bd_vat.ServicePackageID = bd.ServicePackageID
                             AND bd_vat.Amount           < 0) AS TotalDiscount
                 FROM dbo.Invoice i
                 JOIN dbo.BookingDetail bd
                   ON i.ID  = bd.InvoiceID
                 JOIN dbo.m_ServicePackage sp
                   ON sp.ID = bd.ServicePackageID
                WHERE i.SessionID = @sessionId
                  AND bd.RowState = 0) a
     GROUP BY a.ServiceRateID,
              a.ServiceRateName,
              a.Price
     ORDER BY a.ServiceRateName ASC;

--SELECT a.ServiceRateName,
--       a.Quantity,
--       a.Price,
--       a.Amount,
--       a.TotalTax,
--       (a.Amount - a.TotalDiscount) AS TotalMoney,
--       a.TotalDiscount
--  FROM (   SELECT (   SELECT TOP 1 i18n.Title
--                        FROM dbo.m_ServiceRate_i18n i18n
--                       WHERE i18n.ServiceRateID   = sp.ServiceRateID
--                         AND LOWER(i18n.LangCode) = 'vi') AS ServiceRateName,
--                  (bd.Quantity / sp.UnitQuantity) AS Quantity,
--                  bd.Price,
--	  sp.ServiceRateID,
--                  i.CreatedDate,
--                  (   SELECT IIF(SUM(bd_vat.Amount) IS NULL, 0, SUM(bd_vat.Amount))
--                        FROM dbo.BookingDetail bd_vat
--                       WHERE bd_vat.IsSplit          = 0
--                         AND bd_vat.RowState         = 2
--                         AND bd_vat.GroupBy          = bd.GroupBy
--                         AND bd_vat.BookingID        = bd.BookingID
--                         AND bd_vat.ServicePackageID = bd.ServicePackageID
--                         AND bd_vat.Amount           > 0) AS TotalTax,
--                  (bd.Amount) AS Amount,
--                  (   SELECT IIF(SUM(bd_vat.Amount) IS NULL, 0, SUM(bd_vat.Amount))
--                        FROM dbo.BookingDetail bd_vat
--                       WHERE bd_vat.IsSplit          = 0
--                         AND bd_vat.RowState         = 0
--                         AND bd_vat.GroupBy          = bd.GroupBy
--                         AND bd_vat.BookingID        = bd.BookingID
--                         AND bd_vat.ServicePackageID = bd.ServicePackageID
--                         AND bd_vat.Amount           < 0) AS TotalDiscount
--             FROM dbo.Invoice i
--             JOIN dbo.BookingDetail bd
--               ON i.ID  = bd.InvoiceID
--             JOIN dbo.m_ServicePackage sp
--               ON sp.ID = bd.ServicePackageID
--            WHERE i.SessionID            = @sessionId
--              AND bd.RowState            = 0
--              --AND bd.Amount              > 0
--              --AND (   bd.OriginInvoiceID IS NULL
--              --   OR   bd.OriginInvoiceID = @EmptyGuid)
--	 ) a
-- --ORDER BY a.CreatedDate DESC;

END;
GO
/****** Object:  StoredProcedure [dbo].[RptServiceSales]    Script Date: 3/28/2019 1:54:05 PM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date,,>
-- Description:	<Description,,>
-- =============================================
CREATE PROCEDURE [dbo].[RptServiceSales] @sessionId UNIQUEIDENTIFIER = NULL
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE @EmptyGuid UNIQUEIDENTIFIER = '00000000-0000-0000-0000-000000000000';

    SELECT a.ServiceRateName,
           SUM(a.Quantity) AS Quantity,
           a.Price,
           SUM(a.TotalTax) AS TotalTax,
           SUM(a.Amount - a.TotalDiscount) AS TotalMoney,
           SUM(a.TotalDiscount) AS TotalDiscount
      FROM (   SELECT (   SELECT TOP 1 i18n.Title
                            FROM dbo.m_ServiceRate_i18n i18n
                           WHERE i18n.ServiceRateID   = sp.ServiceRateID
                             AND LOWER(i18n.LangCode) = 'vi') AS ServiceRateName,
                      (bd.Quantity / sp.UnitQuantity) AS Quantity,
                      bd.Price,
                      sp.ServiceRateID,
                      i.CreatedDate,
                      (   SELECT IIF(SUM(bd_vat.Amount) IS NULL, 0, SUM(bd_vat.Amount))
                            FROM dbo.BookingDetail bd_vat
                           WHERE bd_vat.IsSplit          = 0
                             AND bd_vat.RowState         = 2
                             AND bd_vat.GroupBy          = bd.GroupBy
                             AND bd_vat.BookingID        = bd.BookingID
                             AND bd_vat.ServicePackageID = bd.ServicePackageID
                             AND bd_vat.Amount           > 0) AS TotalTax,
                      (bd.Amount) AS Amount,
                      (   SELECT IIF(SUM(bd_vat.Amount) IS NULL, 0, SUM(bd_vat.Amount))
                            FROM dbo.BookingDetail bd_vat
                           WHERE bd_vat.IsSplit          = 0
                             AND bd_vat.RowState         = 0
                             AND bd_vat.GroupBy          = bd.GroupBy
                             AND bd_vat.BookingID        = bd.BookingID
                             AND bd_vat.ServicePackageID = bd.ServicePackageID
                             AND bd_vat.Amount           < 0) AS TotalDiscount
                 FROM dbo.Invoice i
                 JOIN dbo.BookingDetail bd
                   ON i.ID  = bd.InvoiceID
                 JOIN dbo.m_ServicePackage sp
                   ON sp.ID = bd.ServicePackageID
                WHERE i.SessionID = @sessionId
                  AND bd.RowState = 0) a
     GROUP BY a.ServiceRateID,
              a.ServiceRateName,
              a.Price
     ORDER BY a.ServiceRateName ASC;

--SELECT a.ServiceRateName,
--       a.Quantity,
--       a.Price,
--       a.Amount,
--       a.TotalTax,
--       (a.Amount - a.TotalDiscount) AS TotalMoney,
--       a.TotalDiscount
--  FROM (   SELECT (   SELECT TOP 1 i18n.Title
--                        FROM dbo.m_ServiceRate_i18n i18n
--                       WHERE i18n.ServiceRateID   = sp.ServiceRateID
--                         AND LOWER(i18n.LangCode) = 'vi') AS ServiceRateName,
--                  (bd.Quantity / sp.UnitQuantity) AS Quantity,
--                  bd.Price,
--                  i.CreatedDate,
--                  (   SELECT IIF(SUM(bd_vat.Amount) IS NULL, 0, SUM(bd_vat.Amount))
--                        FROM dbo.BookingDetail bd_vat
--                       WHERE bd_vat.IsSplit          = 0
--                         AND bd_vat.RowState         = 2
--                         AND bd_vat.GroupBy          = bd.GroupBy
--                         AND bd_vat.BookingID        = bd.BookingID
--                         AND bd_vat.ServicePackageID = bd.ServicePackageID
--                         AND bd_vat.Amount           > 0) AS TotalTax,
--                  (bd.Amount) AS Amount,
--                  (   SELECT IIF(SUM(bd_vat.Amount) IS NULL, 0, SUM(bd_vat.Amount))
--                        FROM dbo.BookingDetail bd_vat
--                       WHERE bd_vat.IsSplit          = 0
--                         AND bd_vat.RowState         = 0
--                         AND bd_vat.GroupBy          = bd.GroupBy
--                         AND bd_vat.BookingID        = bd.BookingID
--                         AND bd_vat.ServicePackageID = bd.ServicePackageID
--                         AND bd_vat.Amount           < 0) AS TotalDiscount
--             FROM dbo.Invoice i
--             JOIN dbo.BookingDetail bd
--               ON i.ID  = bd.InvoiceID
--             JOIN dbo.m_ServicePackage sp
--               ON sp.ID = bd.ServicePackageID
--            WHERE i.SessionID            = @sessionId
--              AND bd.RowState            = 0
--              AND bd.Amount              > 0
--              AND (   bd.OriginInvoiceID IS NULL
--                 OR   bd.OriginInvoiceID = @EmptyGuid)) a
-- ORDER BY a.CreatedDate DESC;

END;
GO
