---------------------------
EasyPoi document
---------------------------
--------------------------
EasyPoi template expression support
--------------------------
Space partitioning
- three mesh operations {{test? Obj:obj2}}
- n: indicates that this cell is a numeric type {{n:}}
- le: ({{le:) - represents the length of}} by {{le: in if/else (obj1: obj2}}) > 8?
- fd: format {{fd: (obj; yyyy-MM-dd) time}}
- fn: format digital {{fn: (obj; ###.00)}}
- fe: traverses data and creates row
- fe: traversal data does not create row
- $fe: move down, insert the current row, move the following rows down.Size (), and then insert
- Delete column - if:! If: (test)}} turning!
- single quotes refer to constant values, such as'1', and then output is 1
- &NULL& control
- "a newline



---------------------------
Export instance of EasyPoi
---------------------------
1.the annotation, import and export are based on annotations, entities on the note, indicating export objects, and can do some operations

```Java
	@ExcelTarget("courseEntity")
	public class CourseEntity implements java.io.Serializable {
	/** 主键 */
	private String id;
	/** 课程名称 */
	@Excel(name = "课程名称", orderNum = "1", needMerge = true)
	private String name;
	/** 老师主键 */
	@ExcelEntity(id = "yuwen")
	@ExcelVerify()
	private TeacherEntity teacher;
	/** 老师主键 */
	@ExcelEntity(id = "shuxue")
	private TeacherEntity shuxueteacher;

	@ExcelCollection(name = "选课学生", orderNum = "4")
	private List<StudentEntity> students;
```

2. basis export
Import export arguments, export objects, and object lists to complete the export

```Java
	HSSFWorkbook workbook = ExcelExportUtil.exportExcel(new ExportParams(
				"2412312", "测试", "测试"), CourseEntity.class, list);
```

3. base export with index
You can add indexes to the exported column when you set a value everywhere

```Java
	ExportParams params = new ExportParams("2412312", "测试", "测试");
	params.setAddIndex(true);
	HSSFWorkbook workbook = ExcelExportUtil.exportExcel(params,
			TeacherEntity.class, telist);
```		
	
4. export Map
Create a similar set of annotations, you can complete the export of Map, a little trouble

```Java
	List<ExcelExportEntity> entity = new ArrayList<ExcelExportEntity>();
	entity.add(new ExcelExportEntity("姓名", "name"));
	entity.add(new ExcelExportEntity("性别", "sex"));

	List<Map<String, String>> list = new ArrayList<Map<String, String>>();
	Map<String, String> map;
	for (int i = 0; i < 10; i++) {
		map = new HashMap<String, String>();
		map.put("name", "1" + i);
		map.put("sex", "2" + i);
		list.add(map);
	}

	HSSFWorkbook workbook = ExcelExportUtil.exportExcel(new ExportParams(
			"测试", "测试"), entity, list);	
```	
		
5. template export
According to the template configuration, the corresponding export is completed

```Java
	TemplateExportParams params = new TemplateExportParams();
	params.setHeadingRows(2);
	params.setHeadingStartRow(2);
	Map<String,Object> map = new HashMap<String, Object>();
    map.put("year", "2013");
    map.put("sunCourses", list.size());
    Map<String,Object> obj = new HashMap<String, Object>();
    map.put("obj", obj);
    obj.put("name", list.size());
	params.setTemplateUrl("org/jeecgframework/poi/excel/doc/exportTemp.xls");
	Workbook book = ExcelExportUtil.exportExcel(params, CourseEntity.class, list,
			map);
```	
		
6. import
Set the import parameter, pass in the file or stream, and then get the corresponding list

```Java
	ImportParams params = new ImportParams();
	params.setTitleRows(2);
	params.setHeadRows(2);
	//params.setSheetNum(9);
	params.setNeedSave(true);
	long start = new Date().getTime();
	List<CourseEntity> list = ExcelImportUtil.importExcel(new File(
			"d:/tt.xls"), CourseEntity.class, params);
```	

7. with spring MVC
Just a few words, Excel export OK
	
```Java
	@RequestMapping(params = "exportXls")
	public String exportXls(CourseEntity course,HttpServletRequest request,HttpServletResponse response
			, DataGrid dataGrid,ModelMap map) {

        CriteriaQuery cq = new CriteriaQuery(CourseEntity.class, dataGrid);
        org.jeecgframework.core.extend.hqlsearch.HqlGenerateUtil.installHql(cq, course, request.getParameterMap());
        List<CourseEntity> courses = this.courseService.getListByCriteriaQuery(cq,false);

        map.put(NormalExcelConstants.FILE_NAME,"用户信息");
        map.put(NormalExcelConstants.CLASS,CourseEntity.class);
        map.put(NormalExcelConstants.PARAMS,new ExportParams("课程列表", "导出人:Jeecg",
                "导出信息"));
        map.put(NormalExcelConstants.DATA_LIST,courses);
        return NormalExcelConstants.JEECG_EXCEL_VIEW;

	}
```

8.Excel imports validation, filters data that does not conform to the rules, adds error information to the Excel, provides commonly used checksum rules, and has common checkout interfaces

```Java
    @Excel(name = "Email", width = 25)
    @Max(value = 15,message = "max 最大值不能超过15")
    private int email;
    /**
     * 手机号
     */
    @Excel(name = "Mobile", width = 20)
    @NotNull
    private String mobile;
    
    ExcelImportResult<ExcelVerifyEntity> result = ExcelImportUtil.importExcelVerify(new File(
            "d:/tt.xls"), ExcelVerifyEntity.class, params);
    for (int i = 0; i < result.getList().size(); i++) {
        System.out.println(ReflectionToStringBuilder.toString(result.getList().get(i)));
    }
```
9. import Map
Set import parameters, incoming files or streams, you can get the corresponding list, custom Key, you need to implement the IExcelDataHandler interface
	
```Java
	ImportParams params = new ImportParams();
	List<Map<String,Object>> list = ExcelImportUtil.importExcel(new File(
			"d:/tt.xls"), Map.class, params);
```	

10. large amount of data Excel export
ExportBigExcel method, you can finally turn off closeExportBigExcel, or you can not close it
	
11., if View does not work, has been found by other View out of the case, the use of the following, carried out a unified package, the same effect

```Java
	PoiBaseView
	public static void render(Map<String, Object> model, HttpServletRequest request,
                              HttpServletResponse response, String viewName) {
        PoiBaseView view = null;
        if (BigExcelConstants.BIG_EXCEL_VIEW.equals(viewName)) {
            view = new BigExcelExportView();
        } else if (MapExcelConstants.JEECG_MAP_EXCEL_VIEW.equals(viewName)) {
            view = new JeecgMapExcelView();
        } else if (NormalExcelConstants.JEECG_EXCEL_VIEW.equals(viewName)) {
            view = new JeecgSingleExcelView();
        } else if (TemplateExcelConstants.JEECG_TEMPLATE_EXCEL_VIEW.equals(viewName)) {
            view = new JeecgTemplateExcelView();
        } else if (MapExcelGraphConstants.MAP_GRAPH_EXCEL_VIEW.equals(viewName)) {
            view = new MapGraphExcelView();
        }
        try {
            view.renderMergedOutputModel(model, request, response);
        } catch (Exception e) {
            LOGGER.error(e.getMessage(), e);
        }
    }
	// Demo
	@RequestMapping(params = "exportXls")
	public void exportXls(CourseEntity course,HttpServletRequest request,HttpServletResponse response
			, DataGrid dataGrid,ModelMap map) {
        CriteriaQuery cq = new CriteriaQuery(CourseEntity.class, dataGrid);
        org.jeecgframework.core.extend.hqlsearch.HqlGenerateUtil.installHql(cq, course, request.getParameterMap());
        List<CourseEntity> courses = this.courseService.getListByCriteriaQuery(cq,false);
        map.put(NormalExcelConstants.FILE_NAME,"用户信息");
        map.put(NormalExcelConstants.CLASS,CourseEntity.class);
        map.put(NormalExcelConstants.PARAMS,new ExportParams("课程列表", "导出人:Jeecg",
                "导出信息"));
        map.put(NormalExcelConstants.DATA_LIST,courses);
        PoiBaseView.render(map,request,response,NormalExcelConstants.JEECG_EXCEL_VIEW);
	}
```	
