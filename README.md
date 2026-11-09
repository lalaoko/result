# result
TABLE_NAME	COLUMN_NAME	DATA_TYPE	IS_NULLABLE
ProjectHistory	HistoryID	int	NO
ProjectHistory	ProjectID	int	NO
ProjectHistory	ActionType	nvarchar	NO
ProjectHistory	Timestamp	datetime2	NO
ProjectHistory	SnapshotCount	int	NO
Projects	ProjectID	int	NO
Projects	Title	nvarchar	NO
Projects	Author	nvarchar	NO
Projects	CreatedAt	datetime2	NO
Projects	UpdatedAt	datetime2	NO
Projects	CanvasWidth	int	NO
Projects	CanvasHeight	int	NO
Shapes	ShapeID	int	NO
Shapes	ProjectID	int	NO
Shapes	ShapeTypeID	int	NO
Shapes	X	int	NO
Shapes	Y	int	NO
Shapes	Width	int	NO
Shapes	Height	int	NO
Shapes	StrokeColor	nvarchar	NO
Shapes	FillColor	nvarchar	NO
Shapes	StrokeWidth	float	NO
Shapes	TextContent	nvarchar	YES
Shapes	ZOrder	int	NO
Shapes	FontSize	real	NO
ShapeTypes	ShapeTypeID	int	NO
ShapeTypes	TypeName	nvarchar	NO
ShapeTypes	DefaultWidth	int	NO
ShapeTypes	DefaultHeight	int	NO
------------------------------
TableName	PrimaryKeyName	ColumnName
ProjectHistory	PK__ProjectH__4D7B4ADDACC54104	HistoryID
Projects	PK__Projects__761ABED0FD409D5A	ProjectID
Shapes	PK__Shapes__70FC83A1DD8A6876	ShapeID
ShapeTypes	PK__ShapeTyp__19D015BF7CDDAA8F	ShapeTypeID
--------------------------------------------
ForeignKeyName	ChildTable	ParentTable	DeleteRule
FK__Shapes__ProjectI__300424B4	Shapes	Projects	CASCADE
FK__Shapes__ShapeTyp__30F848ED	Shapes	ShapeTypes	NO_ACTION
-----------------------------------
ViewName	ViewDefinition
v_ProjectStatistics	    CREATE   VIEW v_ProjectStatistics  AS  SELECT      p.ProjectID AS [ID],      p.Title AS [Название схемы],      p.Author AS [Автор],      p.UpdatedAt AS [Последнее изменение],      COUNT(s.ShapeID) AS [Всего фигур],        ISNULL(          SUM(CASE              WHEN st.TypeName = 'Rhombus' THEN 1              ELSE 0          END), 0      ) AS [Блоков условий],        ISNULL(          SUM(CASE              WHEN st.TypeName = 'TextBlock' THEN 1              ELSE 0          END), 0      ) AS [Текстовых заметок]    FROM Projects p  LEFT JOIN Shapes s      ON p.ProjectID = s.ProjectID  LEFT JOIN ShapeTypes st      ON s.ShapeTypeID = st.ShapeTypeID    GROUP BY      p.ProjectID,      p.Title,      p.Author,      p.UpdatedAt;  ---------------------------------------
ProcedureName	ProcedureDefinition
sp_GetProjectShapes	    -- 2. Процедура загрузки фигур проекта  CREATE   PROCEDURE [dbo].[sp_GetProjectShapes]      @ProjectID INT  AS  BEGIN      SET NOCOUNT ON;        SELECT          s.ShapeID,          s.X,          s.Y,          s.Width,          s.Height,          s.StrokeColor,          s.FillColor,          s.StrokeWidth,          s.TextContent,          s.ZOrder,          s.FontSize,          st.TypeName      FROM Shapes s      INNER JOIN ShapeTypes st          ON s.ShapeTypeID = st.ShapeTypeID      WHERE s.ProjectID = @ProjectID      ORDER BY s.ZOrder ASC;  END;  
sp_SaveProject	  CREATE PROCEDURE [dbo].[sp_SaveProject]      @ProjectID INT OUTPUT,      @Title NVARCHAR(100),      @Author NVARCHAR(50),      @CanvasWidth INT,      @CanvasHeight INT  AS  BEGIN      SET NOCOUNT ON;        IF @ProjectID IS NULL         OR @ProjectID <= 0         OR NOT EXISTS (             SELECT 1             FROM Projects             WHERE ProjectID = @ProjectID         )      BEGIN          INSERT INTO Projects              (Title, Author, CanvasWidth, CanvasHeight)          VALUES              (@Title, @Author, @CanvasWidth, @CanvasHeight);            SET @ProjectID = SCOPE_IDENTITY();      END      ELSE      BEGIN          UPDATE Projects          SET Title = @Title,              Author = @Author,              CanvasWidth = @CanvasWidth,              CanvasHeight = @CanvasHeight,              UpdatedAt = SYSDATETIME()          WHERE ProjectID = @ProjectID;            DELETE FROM Shapes          WHERE ProjectID = @ProjectID;      END  END;  
---------------------------------------------------------------------------------------
TriggerName	ParentTable	IsDisabled	TriggerDefinition
trg_Projects_Audit	Projects	0	  CREATE TRIGGER trg_Projects_Audit  ON Projects  AFTER INSERT, UPDATE, DELETE  AS  BEGIN      SET NOCOUNT ON;        -- Запись создания и обновления      INSERT INTO ProjectHistory          (ProjectID, ActionType, SnapshotCount)      SELECT          i.ProjectID,          CASE              WHEN EXISTS (                  SELECT 1                  FROM deleted d                  WHERE d.ProjectID = i.ProjectID              )              THEN 'UPDATE'              ELSE 'INSERT'          END,          (              SELECT COUNT(*)              FROM Shapes s              WHERE s.ProjectID = i.ProjectID          )      FROM inserted i;        -- Запись удаления      INSERT INTO ProjectHistory          (ProjectID, ActionType, SnapshotCount)      SELECT          d.ProjectID,          'DELETE',          (              SELECT COUNT(*)              FROM Shapes s              WHERE s.ProjectID = d.ProjectID          )      FROM deleted d      WHERE NOT EXISTS (          SELECT 1          FROM inserted i          WHERE i.ProjectID = d.ProjectID      );  END;  
------------------------------------------------------------
ShapeTypeID	TypeName	DefaultWidth	DefaultHeight
1	Rectangle	80	50
2	Ellipse	80	60
3	Rhombus	90	60
4	TextBlock	120	40
5	Line	80	2
------------------------------
ID	Название схемы	Автор	Последнее изменение	Всего фигур	Блоков условий	Текстовых заметок
5	пук	User	2026-11-09 01:41:16.7168798	13	1	0
6	123	User	2026-11-09 01:11:07.5105344	2	1	0
7	проект	User	2026-11-09 01:24:28.9100781	3	0	1
8	Мой проект	User	2026-11-09 01:41:59.5868914	41	0	27
9	098	User	2026-11-09 11:56:21.2374353	1	0	0
10	мааам	User	2026-11-09 12:07:24.0291775	1	0	1
-------------------------------
HistoryID	ProjectID	ActionType	Timestamp	SnapshotCount
31	10	INSERT	2026-11-09 12:07:24.0432419	0
30	9	INSERT	2026-11-09 11:56:21.2483405	0
29	8	UPDATE	2026-11-09 01:41:59.5985156	40
28	8	INSERT	2026-11-09 01:41:53.3134751	0
27	5	UPDATE	2026-11-09 01:41:16.7168798	13
26	5	UPDATE	2026-11-09 01:41:12.1899647	4
25	5	UPDATE	2026-11-09 01:39:40.1192454	3
24	2	DELETE	2026-11-09 01:39:02.4215077	0
23	7	INSERT	2026-11-09 01:24:28.9525963	0
22	6	UPDATE	2026-11-09 01:11:07.5525360	2
21	6	INSERT	2026-11-09 01:09:55.0902154	0
20	5	UPDATE	2026-11-09 01:09:13.9485651	4
19	5	INSERT	2026-11-09 01:09:00.1531340	0
18	1	DELETE	2026-11-09 00:52:30.6659593	0
17	3	DELETE	2026-11-08 21:06:26.4704942	0
15	3	UPDATE	2026-11-08 21:04:09.6804302	3
14	3	UPDATE	2026-11-08 21:03:49.5013450	3
12	3	UPDATE	2026-11-08 21:03:22.7176390	4
11	3	UPDATE	2026-11-08 21:01:08.0781643	6
10	3	UPDATE	2026-11-08 20:56:03.5407230	5
