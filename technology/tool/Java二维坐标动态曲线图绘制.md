# 前言
> 将一串随机数输入到二维坐标轴中，不断刷新JPanel,实现动态显示的效果微笑

# 代码
```java
import java.awt.BasicStroke;
import java.awt.BorderLayout;
import java.awt.Color;
import java.awt.Graphics;
import java.awt.Graphics2D;
import java.awt.RenderingHints;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.Random;
 
import javax.swing.JFrame;
import javax.swing.JPanel;
 
public class Chart_test extends JFrame {
 
	private List<Integer> values;// 保存接受的数据容器
	private static final int MAX_VALUE = 180;// 接受到的数据最大值
	private static final int MAX_COUNT_OF_VALUES = 50;// 最多保存数据的个数
	// private
	private MyCanvas trendChartCanvas = new MyCanvas();
	// 框架起点坐标
	private final int FREAME_X = 50;
	private final int FREAME_Y = 50;
	private final int FREAME_WIDTH = 600;// 横
	private final int FREAME_HEIGHT = 250;// 纵
 
	// 原点坐标
	private final int Origin_X = FREAME_X + 50;
	private final int Origin_Y = FREAME_Y + FREAME_HEIGHT - 30;
 
	// X,Y轴终点坐标
	private final int XAxis_X = FREAME_X + FREAME_WIDTH - 30;
	private final int XAxis_Y = Origin_Y;
	private final int YAxis_X = Origin_X;
	private final int YAxis_Y = FREAME_Y + 30;
 
	// X轴上的时间分度值（1分度=40像素）
	private final int TIME_INTERVAL = 50;
	// Y轴上值
	private final int PRESS_INTERVAL = 30;
 
	public Chart_test() {
		super("前端界面显示：");
		values = Collections.synchronizedList(new ArrayList<Integer>());// 防止引起线程异常
		// 创建一个随机数线程
		new Thread(new Runnable() {
			public void run() {
				Random rand = new Random();
				try {
					while (true) {
						addValue(rand.nextInt(MAX_VALUE) + 90);
						repaint();
						Thread.sleep(100);
					}
				} catch (InterruptedException b) {
					b.printStackTrace();
				}
			}
 
		}).start();
 
		this.setDefaultCloseOperation(EXIT_ON_CLOSE);
		this.setBounds(300, 200, 900, 600);
		this.add(trendChartCanvas, BorderLayout.CENTER);
		this.setVisible(true);
	}
 
	public void addValue(int value) {
		// 循环的使用一个接受数据的空间
		if (values.size() > MAX_COUNT_OF_VALUES) {
			values.remove(0);
		}
		values.add(value);
	}
 
	// 画布重绘图
	class MyCanvas extends JPanel {
		private static final long serialVersionUID = 1L;
 
		public void paintComponent(Graphics g) {
			Graphics2D g2D = (Graphics2D) g;
 
			Color c = new Color(200, 70, 0);
			g.setColor(c);
			super.paintComponent(g);
 
			// 绘制平滑点的曲线
			g2D.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
			int w = XAxis_X;// 起始点
			int xDelta = w / MAX_COUNT_OF_VALUES;
			int length = values.size() - 10;
 
			for (int i = 0; i < length - 1; ++i) {
				g2D.drawLine(xDelta * (MAX_COUNT_OF_VALUES - length + i), values.get(i),
						xDelta * (MAX_COUNT_OF_VALUES - length + i + 1), values.get(i + 1));
			}
			// 画坐标轴
			g2D.setStroke(new BasicStroke(Float.parseFloat("2.0F")));// 轴线粗度
			// X轴以及方向箭头
			g.drawLine(Origin_X, Origin_Y, XAxis_X, XAxis_Y);// x轴线的轴线
			g.drawLine(XAxis_X, XAxis_Y, XAxis_X - 5, XAxis_Y - 5);// 上边箭头
			g.drawLine(XAxis_X, XAxis_Y, XAxis_X + 5, XAxis_Y + 5);// 下边箭头
 
			// Y轴以及方向箭头
			g.drawLine(Origin_X, Origin_Y, YAxis_X, YAxis_Y);
			g.drawLine(YAxis_X, YAxis_Y, YAxis_X - 5, YAxis_Y + 5);
			g.drawLine(YAxis_X, YAxis_Y, YAxis_X + 5, YAxis_Y + 5);
 
			// 画X轴上的时间刻度（从坐标轴原点起，每隔TIME_INTERVAL(时间分度)像素画一时间点，到X轴终点止）
			g.setColor(Color.BLUE);
			g2D.setStroke(new BasicStroke(Float.parseFloat("1.0f")));
 
			// X轴刻度依次变化情况
			for (int i = Origin_X, j = 0; i < XAxis_X; i += TIME_INTERVAL, j += TIME_INTERVAL) {
				g.drawString(" " + j, i - 10, Origin_Y + 20);
			}
			g.drawString("时间", XAxis_X + 5, XAxis_Y + 5);
 
			// 画Y轴上血压刻度（从坐标原点起，每隔10像素画一压力值，到Y轴终点止）
			for (int i = Origin_Y, j = 0; i > YAxis_Y; i -= PRESS_INTERVAL, j += TIME_INTERVAL) {
				g.drawString(j + " ", Origin_X - 30, i + 3);
			}
			g.drawString("幅度/Amplitude", YAxis_X - 5, YAxis_Y - 5);// 血压刻度小箭头值
			// 画网格线
			g.setColor(Color.BLACK);
			// 坐标内部横线
			for (int i = Origin_Y; i > YAxis_Y; i -= PRESS_INTERVAL) {
				g.drawLine(Origin_X, i, Origin_X + 10 * TIME_INTERVAL, i);
			}
			// 坐标内部竖线
			for (int i = Origin_X; i < XAxis_X; i += TIME_INTERVAL) {
				g.drawLine(i, Origin_Y, i, Origin_Y - 6 * PRESS_INTERVAL);
			}
 
		}
	}
 
	public static void main(String[] args) {
		new Chart_test();
	}
}
```