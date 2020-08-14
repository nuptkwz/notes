# 前言
> 将一串随机数输入到二维坐标轴中，不断刷新JPanel,实现动态显示的效果

# 代码
```java
package practice_1;
 
import java.io.BufferedReader;
import java.io.File;
import java.io.FileReader;
 
public class read_txt{
	public static void main(String[] args){
		File file=new File("D:\\aa.txt");
		BufferedReader reader=null;
		String temp=null;
		int line=1;
		try{
		   reader=new BufferedReader(new FileReader(file));
		   while((temp=reader.readLine())!=null){
				System.out.println("line"+line+":"+temp);
				line++;
		   }
		}
		catch(Exception e){
			e.printStackTrace();
		}
		finally{
			if(reader!=null){
				try{
					reader.close();
				}
				catch(Exception e){
					e.printStackTrace();
				}
			}
		}
	}
}
```