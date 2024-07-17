


错误信息

```
Handler dispatch failed; nested exception is java.lang.NoSuchMethodError: org.apache.poi.ss.usermodel.Cell.getCellType()Lorg/apache/poi/ss/usermodel/CellType;
```



查看源码，发现这个地方报错



![OfMAOd](https://xuemingde.com/pages/image/2022/03/08/OfMAOd.png)



poi版本导致（3.17），到时从easypoi中去除poi，重新引用新版本的poi（5.0.0），还是会加载出3.17版本的，未找到原因，最后的解决方法是，重写了easypoi的Excel导入功能。



重写代码：

```Java
public class ExcelUtils {

  	/**
	 *  重写了 easypoi导入，因为easypoi的导入报错
	 * @param inputstream
	 * @param pojoClass
	 * @param params
	 * @param <T>
	 * @return
	 * @throws Exception
	 */
	public static <T> List<T> importExcelRewrite(InputStream inputstream, Class<?> pojoClass, ImportParams params) throws Exception {
		return new ExcelImportServiceImok().importExcelByIss(inputstream, pojoClass, params, false).getList();
	}
}

```

```java

import cn.afterturn.easypoi.entity.BaseTypeConstants;
import cn.afterturn.easypoi.excel.annotation.ExcelTarget;
import cn.afterturn.easypoi.excel.entity.ImportParams;
import cn.afterturn.easypoi.excel.entity.params.ExcelCollectionParams;
import cn.afterturn.easypoi.excel.entity.params.ExcelImportEntity;
import cn.afterturn.easypoi.excel.entity.result.ExcelImportResult;
import cn.afterturn.easypoi.excel.entity.result.ExcelVerifyHandlerResult;
import cn.afterturn.easypoi.excel.imports.ExcelImportService;
import cn.afterturn.easypoi.excel.imports.recursive.ExcelImportForkJoinWork;
import cn.afterturn.easypoi.exception.excel.ExcelImportException;
import cn.afterturn.easypoi.exception.excel.enums.ExcelImportEnum;
import cn.afterturn.easypoi.handler.inter.IExcelDataHandler;
import cn.afterturn.easypoi.handler.inter.IExcelDataModel;
import cn.afterturn.easypoi.handler.inter.IExcelDictHandler;
import cn.afterturn.easypoi.handler.inter.IExcelModel;
import cn.afterturn.easypoi.util.PoiPublicUtil;
import cn.afterturn.easypoi.util.PoiReflectorUtil;
import cn.afterturn.easypoi.util.PoiValidationUtil;
import org.apache.commons.lang3.StringUtils;
import org.apache.commons.lang3.builder.ReflectionToStringBuilder;
import org.apache.commons.lang3.time.DateUtils;
import org.apache.poi.hssf.usermodel.HSSFSheet;
import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.apache.poi.ss.formula.functions.T;
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.util.IOUtils;
import org.apache.poi.xssf.usermodel.XSSFSheet;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.*;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.lang.reflect.Type;
import java.math.BigDecimal;
import java.sql.Time;
import java.sql.Timestamp;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.time.ZoneId;
import java.util.*;
import java.util.concurrent.ForkJoinPool;

/**
 * @author X-MD
 * @version 1.0.0
 * @Description 重写表格导入
 * @createTime 2022年03月08日 14:56:00
 */
public class ExcelImportServiceImok extends ExcelImportService {
	private static final Logger LOGGER = LoggerFactory.getLogger(ExcelImportServiceImok.class);
	private List<String> handlerList = null;

	private boolean verifyFail = false;
	/**
	 * 异常数据styler
	 */
	private CellStyle errorCellStyle;

	private List<Row> successRow;
	private List<Row> failRow;
	private List failCollection;

	public ExcelImportServiceImok() {
		successRow = new ArrayList<>();
		failRow = new ArrayList<>();
		failCollection = new ArrayList();
	}

	/***
	 * 向List里面继续添加元素
	 *
	 * @param object
	 * @param param
	 * @param row
	 * @param titlemap
	 * @param targetId
	 * @param pictures
	 * @param params
	 */
	public void addListContinues(Object object, ExcelCollectionParams param, Row row,
								 Map<Integer, String> titlemap, String targetId,
								 Map<String, PictureData> pictures,
								 ImportParams params, StringBuilder errorMsg) throws Exception {
		Collection collection = (Collection) PoiReflectorUtil.fromCache(object.getClass())
				.getValue(object, param.getName());
		Object entity = PoiPublicUtil.createObject(param.getType(), targetId);
		if (entity instanceof IExcelDataModel) {
			((IExcelDataModel) entity).setRowNum(row.getRowNum());
		}
		String picId;
		// 是否需要加上这个对象
		boolean isUsed = false;
		for (int i = row.getFirstCellNum(); i < titlemap.size(); i++) {
			Cell cell = row.getCell(i);
			String titleString = titlemap.get(i);
			if (param.getExcelParams().containsKey(titleString)) {
				if (param.getExcelParams().get(titleString).getType() == BaseTypeConstants.IMAGE_TYPE) {
					picId = row.getRowNum() + "_" + i;
					saveImage(entity, picId, param.getExcelParams(), titleString, pictures, params);
				} else {
					try {
						saveFieldValues(params, entity, cell, param.getExcelParams(), titleString, row);
					} catch (ExcelImportException e) {
						// 如果需要去校验就忽略,这个错误,继续执行
						if (params.isNeedVerify() && ExcelImportEnum.GET_VALUE_ERROR.equals(e.getType())) {
							errorMsg.append(" ").append(titleString).append(ExcelImportEnum.GET_VALUE_ERROR.getMsg());
						}
					}
				}
				isUsed = true;
			}
		}
		if (isUsed) {
			collection.add(entity);
		}
	}

	/**
	 * 获取key的值,针对不同类型获取不同的值
	 *
	 * @author JueYue 2013-11-21
	 */
	private String getKeyValue(Cell cell) {
		Object obj = PoiCellUtils.getCellValue(cell);
		return obj == null ? null : obj.toString().trim();
	}

	/**
	 * 获取保存的真实路径
	 */
	private String getSaveUrl(ExcelImportEntity excelImportEntity, Object object) throws Exception {
		if (ExcelImportEntity.IMG_SAVE_PATH.equals(excelImportEntity.getSaveUrl())) {
			if (excelImportEntity.getMethods() != null
					&& excelImportEntity.getMethods().size() > 0) {
				object = getFieldBySomeMethod(excelImportEntity.getMethods(), object);
			}
			String url = object.getClass().getName()
					.split("\\.")[object.getClass().getName().split("\\.").length - 1];
			return excelImportEntity.getSaveUrl() + File.separator + url;
		}
		return excelImportEntity.getSaveUrl();
	}

	private <T> List<T> importExcel(Collection<T> result, Sheet sheet, Class<?> pojoClass,
									ImportParams params,
									Map<String, PictureData> pictures) throws Exception {
		List collection = new ArrayList();
		Map<String, ExcelImportEntity> excelParams = new HashMap<>();
		List<ExcelCollectionParams> excelCollection = new ArrayList<>();
		String targetId = null;
		i18nHandler = params.getI18nHandler();
		boolean isMap = Map.class.equals(pojoClass);
		if (!isMap) {
			Field[] fileds = PoiPublicUtil.getClassFields(pojoClass);
			ExcelTarget etarget = pojoClass.getAnnotation(ExcelTarget.class);
			if (etarget != null) {
				targetId = etarget.value();
			}
			getAllExcelField(targetId, fileds, excelParams, excelCollection, pojoClass, null, null);
		}
		Iterator<Row> rows = sheet.rowIterator();
		for (int j = 0; j < params.getTitleRows(); j++) {
			rows.next();
		}
		Map<Integer, String> titlemap = getTitleMap(rows, params, excelCollection, excelParams);
		checkIsValidTemplate(titlemap, excelParams, params, excelCollection);
		Object object = null;
		String picId;
		int readRow = 1;
		//跳过无效行
		for (int i = 0; i < params.getStartRows(); i++) {
			rows.next();
		}
		//判断index 和集合,集合情况默认为第一列
		if (excelCollection.size() > 0 && params.getKeyIndex() == null) {
			params.setKeyIndex(0);
		}
		int endRow = sheet.getLastRowNum() - params.getLastOfInvalidRow();
		if (params.getReadRows() > 0) {
			endRow = Math.min(params.getReadRows(), endRow);
		}
		if (params.isConcurrentTask()) {
			ForkJoinPool forkJoinPool = new ForkJoinPool();
			ExcelImportForkJoinWork task = new ExcelImportForkJoinWork(params.getStartRows() + params.getHeadRows() + params.getTitleRows(), endRow, sheet, params, pojoClass, this, targetId, titlemap, excelParams);
			ExcelImportResult forkJoinResult = forkJoinPool.invoke(task);
			collection = forkJoinResult.getList();
			failCollection = forkJoinResult.getFailList();
		} else {
			StringBuilder errorMsg;
			while (rows.hasNext()) {
				Row row = rows.next();
				// Fix 如果row为无效行时候跳出
				if (row.getRowNum() > endRow) {
					break;
				}
				/* 如果当前行的单元格都是无效的，那就继续下一行 */
				if (row.getLastCellNum() < 0) {
					continue;
				}
				if (isMap && object != null) {
					((Map) object).put("excelRowNum", row.getRowNum());
				}
				errorMsg = new StringBuilder();
				// 判断是集合元素还是不是集合元素,如果是就继续加入这个集合,不是就创建新的对象
				// keyIndex 如果为空就不处理,仍然处理这一行
				if (params.getKeyIndex() != null
						&& (row.getCell(params.getKeyIndex()) == null
						|| StringUtils.isEmpty(getKeyValue(row.getCell(params.getKeyIndex()))))
						&& object != null) {
					for (ExcelCollectionParams param : excelCollection) {
						addListContinues(object, param, row, titlemap, targetId, pictures, params, errorMsg);
					}
				} else {
					object = PoiPublicUtil.createObject(pojoClass, targetId);
					try {
						Set<Integer> keys = titlemap.keySet();
						for (Integer cn : keys) {
							Cell cell = row.getCell(cn);
							String titleString = titlemap.get(cn);
							if (excelParams.containsKey(titleString) || isMap) {
								if (excelParams.get(titleString) != null
										&& excelParams.get(titleString).getType() == BaseTypeConstants.IMAGE_TYPE) {
									picId = row.getRowNum() + "_" + cn;
									saveImage(object, picId, excelParams, titleString, pictures,
											params);
								} else {
									try {
										saveFieldValues(params, object, cell, excelParams, titleString, row);
									} catch (ExcelImportException e) {
										// 如果需要去校验就忽略,这个错误,继续执行
										if (params.isNeedVerify() && ExcelImportEnum.GET_VALUE_ERROR.equals(e.getType())) {
											errorMsg.append(" ").append(titleString).append(ExcelImportEnum.GET_VALUE_ERROR.getMsg());
										}
									}
								}
							}
						}

						if (object instanceof IExcelDataModel) {
							((IExcelDataModel) object).setRowNum(row.getRowNum());
						}
						for (ExcelCollectionParams param : excelCollection) {
							addListContinues(object, param, row, titlemap, targetId, pictures, params, errorMsg);
						}
						if (verifyingDataValiditys(object, row, params, isMap, errorMsg)) {
							collection.add(object);
						} else {
							failCollection.add(object);
						}
					} catch (ExcelImportException e) {
						LOGGER.error("excel import error , row num:{},obj:{}", readRow, ReflectionToStringBuilder.toString(object));
						if (!e.getType().equals(ExcelImportEnum.VERIFY_ERROR)) {
							throw new ExcelImportException(e.getType(), e);
						}
					} catch (Exception e) {
						LOGGER.error("excel import error , row num:{},obj:{}", readRow, ReflectionToStringBuilder.toString(object));
						throw new RuntimeException(e);
					}
				}
				readRow++;
			}
		}
		return collection;
	}

	/**
	 * 校验数据合法性
	 */
	public boolean verifyingDataValiditys(Object object, Row row, ImportParams params,
										  boolean isMap, StringBuilder fieldErrorMsg) {
		boolean isAdd = true;
		Cell cell = null;
		if (params.isNeedVerify()) {
			String errorMsg = PoiValidationUtil.validation(object, params.getVerifyGroup());
			if (StringUtils.isNotEmpty(errorMsg)) {
				cell = row.createCell(row.getLastCellNum());
				cell.setCellValue(errorMsg);
				if (object instanceof IExcelModel) {
					IExcelModel model = (IExcelModel) object;
					model.setErrorMsg(errorMsg);
				}
				isAdd = false;
				verifyFail = true;
			}
		}
		if (params.getVerifyHandler() != null) {
			ExcelVerifyHandlerResult result = params.getVerifyHandler().verifyHandler(object);
			if (!result.isSuccess()) {
				if (cell == null) {
					cell = row.createCell(row.getLastCellNum());
				}
				cell.setCellValue((StringUtils.isNoneBlank(cell.getStringCellValue())
						? cell.getStringCellValue() + "," : "") + result.getMsg());
				if (object instanceof IExcelModel) {
					IExcelModel model = (IExcelModel) object;
					model.setErrorMsg((StringUtils.isNoneBlank(model.getErrorMsg())
							? model.getErrorMsg() + "," : "") + result.getMsg());
				}
				isAdd = false;
				verifyFail = true;
			}
		}
		if ((params.isNeedVerify() || params.getVerifyHandler() != null) && fieldErrorMsg.length() > 0) {
			if (object instanceof IExcelModel) {
				IExcelModel model = (IExcelModel) object;
				model.setErrorMsg((StringUtils.isNoneBlank(model.getErrorMsg())
						? model.getErrorMsg() + "," : "") + fieldErrorMsg.toString());
			}
			if (cell == null) {
				cell = row.createCell(row.getLastCellNum());
			}
			cell.setCellValue((StringUtils.isNoneBlank(cell.getStringCellValue())
					? cell.getStringCellValue() + "," : "") + fieldErrorMsg.toString());
			isAdd = false;
			verifyFail = true;
		}
		if (cell != null) {
			cell.setCellStyle(errorCellStyle);
			failRow.add(row);
			if (isMap) {
				((Map) object).put("excelErrorMsg", cell.getStringCellValue());
			}
		} else {
			successRow.add(row);
		}
		return isAdd;
	}

	/**
	 * 获取表格字段列名对应信息
	 */
	private Map<Integer, String> getTitleMap(Iterator<Row> rows, ImportParams params,
											 List<ExcelCollectionParams> excelCollection,
											 Map<String, ExcelImportEntity> excelParams) {
		Map<Integer, String> titlemap = new LinkedHashMap<>();
		Iterator<Cell> cellTitle;
		String collectionName = null;
		ExcelCollectionParams collectionParams = null;
		Row row = null;
		for (int j = 0; j < params.getHeadRows(); j++) {
			row = rows.next();
			if (row == null) {
				continue;
			}
			cellTitle = row.cellIterator();
			while (cellTitle.hasNext()) {
				Cell cell = cellTitle.next();
				String value = getKeyValue(cell);
				value = value.replace("\n", "");
				int i = cell.getColumnIndex();
				//用以支持重名导入
				if (StringUtils.isNotEmpty(value)) {
					if (titlemap.containsKey(i)) {
						collectionName = titlemap.get(i);
						collectionParams = getCollectionParams(excelCollection, collectionName);
						titlemap.put(i, collectionName + "_" + value);
					} else if (StringUtils.isNotEmpty(collectionName) && collectionParams != null
							&& collectionParams.getExcelParams()
							.containsKey(collectionName + "_" + value)) {
						titlemap.put(i, collectionName + "_" + value);
					} else {
						collectionName = null;
						collectionParams = null;
					}
					if (StringUtils.isEmpty(collectionName)) {
						titlemap.put(i, value);
					}
				}
			}
		}

		// 处理指定列的情况
		Set<String> keys = excelParams.keySet();
		for (String key : keys) {
			if (key.startsWith("FIXED_")) {
				String[] arr = key.split("_");
				titlemap.put(Integer.parseInt(arr[1]), key);
			}
		}
		return titlemap;
	}

	/**
	 * 获取这个名称对应的集合信息
	 */
	private ExcelCollectionParams getCollectionParams(List<ExcelCollectionParams> excelCollection,
													  String collectionName) {
		for (ExcelCollectionParams excelCollectionParams : excelCollection) {
			if (collectionName.equals(excelCollectionParams.getExcelName())) {
				return excelCollectionParams;
			}
		}
		return null;
	}

	/**
	 * Excel 导入 field 字段类型 Integer,Long,Double,Date,String,Boolean
	 */
	public ExcelImportResult importExcelByIss(InputStream inputstream, Class<?> pojoClass,
											  ImportParams params, boolean needMore) throws Exception {
		if (LOGGER.isDebugEnabled()) {
			LOGGER.debug("Excel import start ,class is {}", pojoClass);
		}
		List<T> result = new ArrayList<>();
		ByteArrayOutputStream baos = new ByteArrayOutputStream();
		ExcelImportResult importResult;
		try {
			byte[] buffer = new byte[1024];
			int len;
			while ((len = inputstream.read(buffer)) > -1) {
				baos.write(buffer, 0, len);
			}
			baos.flush();

			InputStream userIs = new ByteArrayInputStream(baos.toByteArray());
			if (LOGGER.isDebugEnabled()) {
				LOGGER.debug("Excel clone success");
			}
			Workbook book = WorkbookFactory.create(userIs);

			boolean isXSSFWorkbook = !(book instanceof HSSFWorkbook);
			if (LOGGER.isDebugEnabled()) {
				LOGGER.debug("Workbook create success");
			}
			importResult = new ExcelImportResult();
			createErrorCellStyle(book);
			Map<String, PictureData> pictures;
			for (int i = params.getStartSheetIndex(); i < params.getStartSheetIndex()
					+ params.getSheetNum(); i++) {
				if (LOGGER.isDebugEnabled()) {
					LOGGER.debug(" start to read excel by is ,startTime is {}", new Date());
				}
				if (isXSSFWorkbook) {
					pictures = PoiPublicUtil.getSheetPictrues07((XSSFSheet) book.getSheetAt(i),
							(XSSFWorkbook) book);
				} else {
					pictures = PoiPublicUtil.getSheetPictrues03((HSSFSheet) book.getSheetAt(i),
							(HSSFWorkbook) book);
				}
				if (LOGGER.isDebugEnabled()) {
					LOGGER.debug(" end to read excel by is ,endTime is {}", new Date());
				}
				result.addAll(importExcel(result, book.getSheetAt(i), pojoClass, params, pictures));
				if (LOGGER.isDebugEnabled()) {
					LOGGER.debug(" end to read excel list by sheet ,endTime is {}", new Date());
				}
				if (params.isReadSingleCell()) {
					readSingleCell(importResult, book.getSheetAt(i), params);
					if (LOGGER.isDebugEnabled()) {
						LOGGER.debug(" read Key-Value ,endTime is {}", System.currentTimeMillis());
					}
				}
			}
			if (params.isNeedSave()) {
				saveThisExcel(params, pojoClass, isXSSFWorkbook, book);
			}
			importResult.setList(result);
			if (needMore) {
				InputStream successIs = new ByteArrayInputStream(baos.toByteArray());
				try {
					Workbook successBook = WorkbookFactory.create(successIs);
					if (params.isVerifyFileSplit()) {
						importResult.setWorkbook(removeSuperfluousRows(successBook, failRow, params));
						importResult.setFailWorkbook(removeSuperfluousRows(book, successRow, params));
					} else {
						importResult.setWorkbook(book);
					}
					importResult.setFailList(failCollection);
					importResult.setVerifyFail(verifyFail);
				} finally {
					successIs.close();
				}
			}
		} finally {
			IOUtils.closeQuietly(baos);
		}

		return importResult;
	}

	private Workbook removeSuperfluousRows(Workbook book, List<Row> rowList, ImportParams params) {
		for (int i = params.getStartSheetIndex(); i < params.getStartSheetIndex()
				+ params.getSheetNum(); i++) {
			for (int j = rowList.size() - 1; j >= 0; j--) {
				if (rowList.get(j).getRowNum() < rowList.get(j).getSheet().getLastRowNum()) {
					book.getSheetAt(i).shiftRows(rowList.get(j).getRowNum() + 1, rowList.get(j).getSheet().getLastRowNum(), -1);
				} else if (rowList.get(j).getRowNum() == rowList.get(j).getSheet().getLastRowNum()) {
					book.getSheetAt(i).createRow(rowList.get(j).getRowNum() + 1);
					book.getSheetAt(i).shiftRows(rowList.get(j).getRowNum() + 1, rowList.get(j).getSheet().getLastRowNum() + 1, -1);
				}
			}
		}
		return book;
	}

	/**
	 * 按照键值对的方式取得Excel里面的数据
	 */
	private void readSingleCell(ExcelImportResult result, Sheet sheet, ImportParams params) {
		if (result.getMap() == null) {
			result.setMap(new HashMap<String, Object>());
		}
		for (int i = 0; i < params.getTitleRows() + params.getHeadRows() + params.getStartRows(); i++) {
			getSingleCellValueForRow(result, sheet.getRow(i), params);
		}

		for (int i = sheet.getLastRowNum() - params.getLastOfInvalidRow(); i < sheet.getLastRowNum(); i++) {
			getSingleCellValueForRow(result, sheet.getRow(i), params);

		}
	}

	private void getSingleCellValueForRow(ExcelImportResult result, Row row, ImportParams params) {
		for (int j = row.getFirstCellNum(), le = row.getLastCellNum(); j < le; j++) {
			String text = PoiCellUtils.getCellValue(row.getCell(j));
			if (StringUtils.isNoneBlank(text) && text.endsWith(params.getKeyMark())) {
				if (result.getMap().containsKey(text)) {
					if (result.getMap().get(text) instanceof String) {
						List<String> list = new ArrayList<>();
						list.add((String) result.getMap().get(text));
						result.getMap().put(text, list);
					}
					((List) result.getMap().get(text)).add(PoiCellUtils.getCellValue(row.getCell(++j)));
				} else {
					result.getMap().put(text, PoiCellUtils.getCellValue(row.getCell(++j)));
				}

			}

		}
	}

	/**
	 * 检查是不是合法的模板
	 */
	private void checkIsValidTemplate(Map<Integer, String> titlemap,
									  Map<String, ExcelImportEntity> excelParams,
									  ImportParams params,
									  List<ExcelCollectionParams> excelCollection) {

		if (params.getImportFields() != null) {
			// 同时校验列顺序
			if (params.isNeedCheckOrder()) {

				if (params.getImportFields().length != titlemap.size()) {
					LOGGER.error("excel列顺序不一致");
					throw new ExcelImportException(ExcelImportEnum.IS_NOT_A_VALID_TEMPLATE);
				}
				int i = 0;
				for (String title : titlemap.values()) {
					if (!StringUtils.equals(title, params.getImportFields()[i++])) {
						LOGGER.error("excel列顺序不一致");
						throw new ExcelImportException(ExcelImportEnum.IS_NOT_A_VALID_TEMPLATE);
					}
				}
			} else {
				for (int i = 0, le = params.getImportFields().length; i < le; i++) {
					if (!titlemap.containsValue(params.getImportFields()[i])) {
						throw new ExcelImportException(ExcelImportEnum.IS_NOT_A_VALID_TEMPLATE);
					}
				}
			}
		} else {
			Collection<ExcelImportEntity> collection = excelParams.values();
			for (ExcelImportEntity excelImportEntity : collection) {
				if (excelImportEntity.isImportField()
						&& !titlemap.containsValue(excelImportEntity.getName())) {
					LOGGER.error(excelImportEntity.getName() + "必须有,但是没找到");
					throw new ExcelImportException(ExcelImportEnum.IS_NOT_A_VALID_TEMPLATE);
				}
			}

			for (int i = 0, le = excelCollection.size(); i < le; i++) {
				ExcelCollectionParams collectionparams = excelCollection.get(i);
				collection = collectionparams.getExcelParams().values();
				for (ExcelImportEntity excelImportEntity : collection) {
					if (excelImportEntity.isImportField() && !titlemap.containsValue(
							collectionparams.getExcelName() + "_" + excelImportEntity.getName())) {
						throw new ExcelImportException(ExcelImportEnum.IS_NOT_A_VALID_TEMPLATE);
					}
				}
			}
		}
	}

	/**
	 * 保存字段值(获取值,校验值,追加错误信息)
	 */
	public void saveFieldValues(ImportParams params, Object object, Cell cell,
								Map<String, ExcelImportEntity> excelParams, String titleString,
								Row row) throws Exception {
		Object value = getValues(params.getDataHandler(), object, cell, excelParams,
				titleString, params.getDictHandler());
		if (object instanceof Map) {
			if (params.getDataHandler() != null) {
				params.getDataHandler().setMapValue((Map) object, titleString, value);
			} else {
				((Map) object).put(titleString, value);
			}
		} else {
			setValues(excelParams.get(titleString), object, value);
		}
	}


	public Object getValues(IExcelDataHandler<?> dataHandler, Object object, Object cell,
							Map<String, ExcelImportEntity> excelParams,
							String titleString, IExcelDictHandler dictHandler) throws Exception {

		ExcelImportEntity entity = excelParams.get(titleString);
		String classFullName = "class java.lang.Object";
		Class clazz = null;
		if (!(object instanceof Map)) {
			Method setMethod = entity.getMethods() != null && entity.getMethods().size() > 0
					? entity.getMethods().get(entity.getMethods().size() - 1) : entity.getMethod();
			Type[] ts = setMethod.getGenericParameterTypes();
			classFullName = ts[0].toString();
			clazz = (Class) ts[0];
		}
		Object result = null;
		if (cell instanceof Cell) {
			result = getCellValues(classFullName, (Cell) cell, entity);
		} else {
			result = cell;
		}
		if (entity != null) {
			result = handlerSuffix(entity.getSuffix(), result);
			result = replaceValue(entity.getReplace(), result);
			result = replaceValue(entity.getReplace(), result);
			if (dictHandler != null && StringUtils.isNoneBlank(entity.getDict())) {
				result = dictHandler.toValue(entity.getDict(), object, entity.getName(), result);
			}
		}
		result = handlerValue(dataHandler, object, result, titleString);
		return getValueByType(classFullName, result, entity, clazz);
	}

	private Object getValueByType(String classFullName, Object result, ExcelImportEntity entity, Class clazz) {
		try {
			//过滤空和空字符串,如果基本类型null会在上层抛出,这里就不处理了
			if (result == null || StringUtils.isBlank(result.toString())) {
				return null;
			}
			if ("class java.util.Date".equals(classFullName) && result instanceof String) {
				return DateUtils.parseDate(result.toString(), entity.getFormat());
			}
			if ("class java.lang.Boolean".equals(classFullName) || "boolean".equals(classFullName)) {
				return Boolean.valueOf(String.valueOf(result));
			}
			if ("class java.lang.Double".equals(classFullName) || "double".equals(classFullName)) {
				return Double.valueOf(String.valueOf(result));
			}
			if ("class java.lang.Long".equals(classFullName) || "long".equals(classFullName)) {
				try {
					return Long.valueOf(String.valueOf(result));
				} catch (Exception e) {
					//格式错误的时候,就用double,然后获取Int值
					return Double.valueOf(String.valueOf(result)).longValue();
				}
			}
			if ("class java.lang.Float".equals(classFullName) || "float".equals(classFullName)) {
				return Float.valueOf(String.valueOf(result));
			}
			if ("class java.lang.Integer".equals(classFullName) || "int".equals(classFullName)) {
				try {
					return Integer.valueOf(String.valueOf(result));
				} catch (Exception e) {
					//格式错误的时候,就用double,然后获取Int值
					return Double.valueOf(String.valueOf(result)).intValue();
				}
			}
			if ("class java.math.BigDecimal".equals(classFullName)) {
				return new BigDecimal(String.valueOf(result));
			}
			if ("class java.lang.String".equals(classFullName)) {
				//针对String 类型,但是Excel获取的数据却不是String,比如Double类型,防止科学计数法
				if (result instanceof String) {
					return result;
				}
				// double类型防止科学计数法
				if (result instanceof Double) {
					return PoiPublicUtil.doubleToString((Double) result);
				}
				return String.valueOf(result);
			}
			if (clazz != null && clazz.isEnum()) {
				if (StringUtils.isNotEmpty(entity.getEnumImportMethod())) {
					return PoiReflectorUtil.fromCache(clazz).execEnumStaticMethod(entity.getEnumImportMethod(), result);
				} else {
					return Enum.valueOf(clazz, result.toString());
				}
			}
			return result;
		} catch (Exception e) {
			LOGGER.error(e.getMessage(), e);
			throw new ExcelImportException(ExcelImportEnum.GET_VALUE_ERROR);
		}
	}

	private Object handlerValue(IExcelDataHandler dataHandler, Object object, Object result,
								String titleString) {
		if (dataHandler == null || dataHandler.getNeedHandlerFields() == null
				|| dataHandler.getNeedHandlerFields().length == 0) {
			return result;
		}
		if (handlerList == null) {
			handlerList = Arrays.asList(dataHandler.getNeedHandlerFields());
		}
		if (handlerList.contains(titleString)) {
			return dataHandler.importHandler(object, titleString, result);
		}
		return result;
	}


	private Object handlerSuffix(String suffix, Object result) {
		if (StringUtils.isNotEmpty(suffix) && result != null
				&& result.toString().endsWith(suffix)) {
			String temp = result.toString();
			return temp.substring(0, temp.length() - suffix.length());
		}
		return result;
	}

	private Object replaceValue(String[] replace, Object result) {
		if (replace != null && replace.length > 0) {
			String temp = String.valueOf(result);
			String[] tempArr;
			for (int i = 0; i < replace.length; i++) {
				tempArr = replace[i].split("_");
				if (temp.equals(tempArr[0])) {
					return tempArr[1];
				}
			}
		}
		return result;
	}

	private Object getCellValues(String classFullName, Cell cell, ExcelImportEntity entity) {
		if (cell == null) {
			return "";
		}
		Object result = null;
		if ("class java.util.Date".equals(classFullName)
				|| "class java.sql.Date".equals(classFullName)
				|| ("class java.sql.Time").equals(classFullName)
				|| ("class java.time.Instant").equals(classFullName)
				|| ("class java.time.LocalDate").equals(classFullName)
				|| ("class java.time.LocalDateTime").equals(classFullName)
				|| ("class java.sql.Timestamp").equals(classFullName)) {
			//FIX: 单元格yyyyMMdd数字时候使用 cell.getDateCellValue() 解析出的日期错误
			if (CellType.NUMERIC == cell.getCellTypeEnum() && DateUtil.isCellDateFormatted(cell)) {
				result = DateUtil.getJavaDate(cell.getNumericCellValue());
			} else {
				String val = "";
				try {
					val = cell.getStringCellValue();
				} catch (Exception e) {
					return null;
				}

				result = getDateData(entity, val);
				if (result == null) {
					return null;
				}
			}
			if (("class java.time.Instant").equals(classFullName)) {
				result = ((Date) result).toInstant();
			} else if (("class java.time.LocalDate").equals(classFullName)) {
				result = ((Date) result).toInstant().atZone(ZoneId.systemDefault()).toLocalDate();
			} else if (("class java.time.LocalDateTime").equals(classFullName)) {
				result = ((Date) result).toInstant().atZone(ZoneId.systemDefault()).toLocalDateTime();
			} else if (("class java.time.OffsetDateTime").equals(classFullName)) {
				result = ((Date) result).toInstant().atZone(ZoneId.systemDefault()).toOffsetDateTime();
			} else if (("class java.time.ZonedDateTime").equals(classFullName)) {
				result = ((Date) result).toInstant().atZone(ZoneId.systemDefault());
			} else if (("class java.sql.Date").equals(classFullName)) {
				result = new java.sql.Date(((Date) result).getTime());
			} else if (("class java.sql.Time").equals(classFullName)) {
				result = new Time(((Date) result).getTime());
			} else if (("class java.sql.Timestamp").equals(classFullName)) {
				result = new Timestamp(((Date) result).getTime());
			}
		} else {
			switch (cell.getCellTypeEnum()) {
				case STRING:
					result = cell.getRichStringCellValue() == null ? ""
							: cell.getRichStringCellValue().getString();
					break;
				case NUMERIC:
					if (DateUtil.isCellDateFormatted(cell)) {
						if ("class java.lang.String".equals(classFullName)) {
							result = formateDate(entity, cell.getDateCellValue());
						}
					} else {
						result = readNumericCell(cell);
					}
					break;
				case BOOLEAN:
					result = Boolean.toString(cell.getBooleanCellValue());
					break;
				case BLANK:
					break;
				case ERROR:
					break;
				case FORMULA:
					try {
						result = readNumericCell(cell);
					} catch (Exception e1) {
						try {
							result = cell.getRichStringCellValue() == null ? ""
									: cell.getRichStringCellValue().getString();
						} catch (Exception e2) {
							throw new RuntimeException("获取公式类型的单元格失败", e2);
						}
					}
					break;
				default:
					break;
			}
		}
		return result;
	}

	private Object readNumericCell(Cell cell) {
		Object result = null;
		double value = cell.getNumericCellValue();
		if (((int) value) == value) {
			result = (int) value;
		} else {
			result = value;
		}
		return result;
	}

	private String formateDate(ExcelImportEntity entity, Date value) {
		if (StringUtils.isNotEmpty(entity.getFormat()) && value != null) {
			SimpleDateFormat format = new SimpleDateFormat(entity.getFormat());
			return format.format(value);
		}
		return null;
	}

	private Date getDateData(ExcelImportEntity entity, String value) {
		if (StringUtils.isNotEmpty(entity.getFormat()) && StringUtils.isNotEmpty(value)) {
			SimpleDateFormat format = new SimpleDateFormat(entity.getFormat());
			try {
				return format.parse(value);
			} catch (ParseException e) {
				try {
					return DateUtil.getJavaDate(Double.parseDouble(value));
				} catch (NumberFormatException ex) {
					LOGGER.error("时间格式化失败,格式化:{},值:{}", entity.getFormat(), value);
					throw new ExcelImportException(ExcelImportEnum.GET_VALUE_ERROR);
				}
			}
		}
		return null;
	}

	/**
	 * @param object
	 * @param picId
	 * @param excelParams
	 * @param titleString
	 * @param pictures
	 * @param params
	 * @throws Exception
	 */
	private void saveImage(Object object, String picId, Map<String, ExcelImportEntity> excelParams,
						   String titleString, Map<String, PictureData> pictures,
						   ImportParams params) throws Exception {
		if (pictures == null) {
			return;
		}
		PictureData image = pictures.get(picId);
		if (image == null) {
			return;
		}
		byte[] data = image.getData();
		String fileName = "pic" + Math.round(Math.random() * 100000000000L);
		fileName += "." + PoiPublicUtil.getFileExtendName(data);
		if (excelParams.get(titleString).getSaveType() == 1) {
			String path = getSaveUrl(excelParams.get(titleString), object);
			File savefile = new File(path);
			if (!savefile.exists()) {
				savefile.mkdirs();
			}
			savefile = new File(path + File.separator + fileName);
			FileOutputStream fos = new FileOutputStream(savefile);
			try {
				fos.write(data);
			} finally {
				IOUtils.closeQuietly(fos);
			}
			setValues(excelParams.get(titleString), object,
					getSaveUrl(excelParams.get(titleString), object) + File.separator + fileName);
		} else {
			setValues(excelParams.get(titleString), object, data);
		}
	}

	private void createErrorCellStyle(Workbook workbook) {
		errorCellStyle = workbook.createCellStyle();
		Font font = workbook.createFont();
		font.setColor(Font.COLOR_RED);
		errorCellStyle.setFont(font);
	}

}

```



调用：

```java
@ApiOperation("导入Excel模板，修改更新数据")
    @PostMapping("/importExcelTemplate")
    public R importMemberList(@RequestPart("file") MultipartFile file) {
        try {
            ImportParams params = new ImportParams();
            params.setTitleRows(1);
            params.setHeadRows(1);
            List<TemplateExcelExport> list = ExcelUtils.importExcelRewrite(file.getInputStream(), TemplateExcelExport.class,params);
            List<AlarmSiteInfo> alarmSiteInfos = BeanMapper.INSTANCE.toAlarmSiteInfoList(list);
            this.alarmSiteService.updateBatchById(alarmSiteInfos);
            return R.ok();
        } catch (Exception e) {
            e.printStackTrace();
            return R.failed("导入失败！");
        }
    }
```

