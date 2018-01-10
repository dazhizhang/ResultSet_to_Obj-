# ResultSet_to_Obj-
Java ResultSet Object
参照：
http://blog.csdn.net/suifeng3051/article/details/39025053
赋值的部分有所修改，把 BeanUtils.setProperty改成了直接的反射机制

package com.

import java.lang.reflect.Field;  
import java.sql.ResultSet;  
import java.sql.ResultSetMetaData;  
import java.sql.SQLException;  
import java.util.ArrayList;  
import java.util.List;  
import javax.persistence.Column;  
import javax.persistence.Entity;  
  
public class ResultSetMapper<T> {  
	
//    @SuppressWarnings("unchecked")
	public List<T> mapRersultSetToObject(ResultSet rs, Class<T> outputClass) {  
        List<T> outputList = null;  
        try {  
            // make sure resultset is not null  
            if (rs != null) {  
                // check if outputClass has 'Entity' annotation  
                if (outputClass.isAnnotationPresent(Entity.class)) {  
                    // get the resultset metadata  
                    ResultSetMetaData rsmd = rs.getMetaData();  
                    // get all the attributes of outputClass  
                    Field[] fields = outputClass.getDeclaredFields();  
                    while (rs.next()) {  
                        T bean = (T) outputClass.newInstance();  
                        for (int _iterator = 0; _iterator < rsmd.getColumnCount(); _iterator++) {  
                            // getting the SQL column name  
                            String columnName = rsmd.getColumnName(_iterator + 1);  
                            // reading the value of the SQL column  
                            Object columnValue = rs.getObject(_iterator + 1);  
                            // iterating over outputClass attributes to check if  
                            // any attribute has 'Column' annotation with  
                            // matching 'name' value  
                            for (Field field : fields) {  
                                if (field.isAnnotationPresent(Column.class)) {  
                                    Column column = field.getAnnotation(Column.class);  
                                    if (column.name().equalsIgnoreCase(columnName) && columnValue != null) {  
//                                      BeanUtils.setProperty(bean, field.getName(), columnValue);  
                                        Field name=bean.getClass().getDeclaredField(columnName);   
                                        name.set(bean, columnValue);     
                                        break;  
                                    }  
                                }  
                            }  
                        }  
                        if (outputList == null) {  
                            outputList = new ArrayList<T>();  
                        }  
                        outputList.add(bean);  
                    }  
  
                } else {  
                    // throw some error  
                }  
            } else {  
                return null;  
            }  
        } catch (IllegalAccessException e) {  
            e.printStackTrace();  
        } catch (SQLException e) {  
            e.printStackTrace();  
        } catch (InstantiationException e) {  
            e.printStackTrace();  
        } catch (NoSuchFieldException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (SecurityException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}  
        return outputList;  
    }  
}  
