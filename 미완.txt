package growfish;

import java.awt.*;
import java.awt.event.*;
import javax.swing.*;
import javax.swing.event.*;
import java.util.*;

public class growfishgame extends JFrame
{
	gamepanel gp;

	growfishgame()
	{
		this.setTitle("물고기 키우기 게임");
		this.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

		gp = new gamepanel();
		this.setContentPane(gp);

		this.setSize(1000, 600);
		this.setVisible(true);
	}

	class gamepanel extends JPanel
	{
		JLabel myfish;
		int myfishwidth = 50; // 기본 물고기 크기
		int myfishheight = 25;
		int dx = 1;
		int dy = 1;
		fishmovethread th = new fishmovethread();
		ArrayList<enemyfishcrowd> enemyfish = new ArrayList<>();
		int enemynum = 20;
		ImageIcon img = new ImageIcon("nimo.png");
		Image ima = img.getImage();
		ImageIcon newimg = new ImageIcon(ima.getScaledInstance(myfishwidth, myfishheight, Image.SCALE_SMOOTH));
		JLabel gameover;
		int gameendcount = 0;

		public gamepanel()
		{
			this.setLayout(null);
			myfish = new JLabel();

			myfish.setIcon(newimg);
			myfish.setSize(myfishwidth, myfishheight);
			myfish.setLocation(800, 300);
			myfish.setOpaque(false);
			myfish.setVisible(true);

			gameover = new JLabel("game over");
			gameover.setFont(new Font("gothic", 200, 200));
			gameover.setLocation(0, 0);
			gameover.setSize(1000, 600);
			gameover.setForeground(Color.white);
			gameover.setVisible(false);
			this.add(gameover);

			for (int i = 0; i < enemynum; i++) // 적물고기 만들기
			{
				int ranx = (int) (Math.random() * 550);
				int rany = (int) (Math.random() * 600);
				int ranwidth = (int) (Math.random() * 350 + 10);
				int ranheight = ranwidth / 3;

				ImageIcon img2 = new ImageIcon("enemy.png");
				Image ima2 = img2.getImage();
				// 사진 여백이 많아서 조금 더 크게 그리기
				ImageIcon newimg2 = new ImageIcon(
						ima2.getScaledInstance(ranwidth * 11 / 10, ranheight * 15 / 10, Image.SCALE_SMOOTH));

				enemyfishcrowd a = new enemyfishcrowd();
				a.setIcon(newimg2);
				a.setSize(ranwidth, ranheight);
				a.setLocation(ranx, rany);
				a.setOpaque(false);
				a.setVisible(true);
				enemyfish.add(a);
				this.add(a);
			}
			this.add(myfish);
			this.addKeyListener(new keyfishmove());
			this.setFocusable(true);

		}

		public void paintComponent(Graphics g)
		{
			ImageIcon bgimg = new ImageIcon("bg.jpg");
			Image bgima = bgimg.getImage();
			ImageIcon bgnewimg = new ImageIcon(bgima.getScaledInstance(1000, 600, Image.SCALE_SMOOTH));

			g.drawImage(bgnewimg.getImage(), 0, 0, null);
			setOpaque(false);
			super.paintComponent(g);
		}

		public void fishmove() // 물고기 항상 같은방향 점점 빨라지면서 이동
		{
			if (myfish.getX() > 1000) // 오른쪽 밖으로 나가면 처음으로
				myfish.setLocation(0, myfish.getY());
			int myx = (int) (Math.random() * 2); // 적속도
			int myy = (int) (Math.random() * -2);
			int myup = (int) (Math.random() * 2);
			int mydown = (int) (Math.random() * -2);
			myfish.setLocation(myfish.getX() + dx + myx + myy, myfish.getY() + dy + myup + mydown);

			for (JLabel efish : enemyfish)
			{
				int enespeed = (int) (Math.random() * 6); // 적속도
				int benespeed = (int) (Math.random() * -4);
				int upenespeed = (int) (Math.random() * 2);
				int downenespeed = (int) (Math.random() * -2);
				int rany = (int) (Math.random() * 600);
				if (efish.getX() > 1100) // x값 넘으면 다시 크기 조절
				{
					int ranwidth = (int) (Math.random() * 350 + 10);
					int ranheight = ranwidth / 3;

					ImageIcon img2 = new ImageIcon("enemy.png");
					Image ima2 = img2.getImage();
					// 사진 여백이 많아서 조금 더 크게 그리기
					ImageIcon newimg2 = new ImageIcon(
							ima2.getScaledInstance(ranwidth * 11 / 10, ranheight * 15 / 10, Image.SCALE_SMOOTH));

					efish.setIcon(newimg2);
					efish.setLocation(-200, rany);
					efish.setSize(ranwidth, ranheight);
				}
				efish.setLocation(efish.getX() + enespeed + benespeed, efish.getY() + upenespeed + downenespeed);
			}

		}

		public boolean contains(JLabel p, int x, int y) // 포함되는지 확인
		{
			if (((p.getX() <= x) && (x < p.getX() + p.getWidth())) && (p.getY() < y) && (y < p.getY() + p.getHeight()))
			{
				// System.out.println(ball.x + " , " + ball.y);
				return true;
			} else
				return false;
		}

		synchronized public void eating()
		{
			for (enemyfishcrowd b : enemyfish)
			{
				// 내 물고기가 적물고기에 닿은지 확인
				if (b.enemyalive && (contains(b, myfish.getX(), myfish.getY())
						|| contains(b, myfish.getX() + myfish.getWidth(), myfish.getY())
						|| contains(b, myfish.getX(), myfish.getY() + myfish.getHeight())
						|| contains(b, myfish.getX() + myfish.getWidth(), myfish.getY() + myfish.getHeight())))
				{
					// 물고기 크기비교
					if (b.getWidth() * b.getHeight() < myfishwidth * myfishheight)
					{
						System.out.println("e : " + b.getWidth() * b.getHeight() + "my : "
								+ myfish.getWidth() * myfish.getHeight());
						System.out.println("fish size" + myfish.getWidth() + " " + myfish.getHeight());
						b.enemyalive = false; // 죽음

						b.setVisible(false); // 지우기
						this.remove(b);
						gameendcount++;

						myfish.setSize(myfishwidth, myfishheight);
						ImageIcon newimg = new ImageIcon(
								ima.getScaledInstance(myfishwidth, myfishheight, Image.SCALE_SMOOTH));

						myfish.setIcon(newimg); // 이미지 크기변경
						myfishwidth += 30; // 물고기 사이즈 변경
						myfishheight += 10;

						System.out.println(gameendcount);
						this.revalidate();
						this.repaint();
					} else // 내 물고기가 더 작으면 게임끝
					{
						gameover.setVisible(true);
					}
				}
				// 적 적물고기가 내 물고기에 닿았는지 확인
				if (b.enemyalive
						&& (contains(myfish, b.getX(), b.getY()) || contains(myfish, b.getX() + b.getWidth(), b.getY())
								|| contains(myfish, b.getX(), b.getY() + b.getHeight())
								|| contains(myfish, b.getX() + b.getWidth(), b.getY() + b.getHeight())))
				{
					// 물고기 크기비교
					if (b.getWidth() * b.getHeight() < myfishwidth * myfishheight)
					{
						System.out.println("e : " + b.getWidth() * b.getHeight() + "my : "
								+ myfish.getWidth() * myfish.getHeight());
						System.out.println("fish size" + myfish.getWidth() + " " + myfish.getHeight());

						b.enemyalive = false; // 죽음

						b.setVisible(false); // 지우기
						this.remove(b);
						gameendcount++;

						myfish.setSize(myfishwidth, myfishheight);
						ImageIcon newimg = new ImageIcon(
								ima.getScaledInstance(myfishwidth, myfishheight, Image.SCALE_SMOOTH));

						myfish.setIcon(newimg); // 이미지 크기변경
						myfishwidth += 30; // 물고기 사이즈 변경
						myfishheight += 10;

						System.out.println(gameendcount);
						this.revalidate();
						this.repaint();
					} else // 내꺼가 더 작으면 게임끝
					{
						gameover.setVisible(true);
					}
				}
			}
			if (gameendcount == 20) // 물고기 다먹었을 때 게임 끝
			{
				gameover.setText("win!!!!");
				gameover.setVisible(true);
			}

		}

		class keyfishmove extends KeyAdapter // 물고기 움직이기
		{

			public void keyPressed(KeyEvent arg0)
			{
				if (arg0.getKeyCode() == KeyEvent.VK_ENTER)
				{
					System.out.println("enter");
					th.start();
				}

				if (arg0.getKeyCode() == KeyEvent.VK_LEFT)
				{
					dx = -1;
					dy = 0;
					myfish.setLocation(myfish.getX() + dx, myfish.getY() + dy);
				}

				if (arg0.getKeyCode() == KeyEvent.VK_RIGHT)
				{
					dx = 1;
					dy = 0;
					myfish.setLocation(myfish.getX() + dx, myfish.getY() + dy);
				}
				if (arg0.getKeyCode() == KeyEvent.VK_UP)
				{
					dx = 0;
					dy = -1;
					myfish.setLocation(myfish.getX() + dx, myfish.getY() + dy);
				}

				if (arg0.getKeyCode() == KeyEvent.VK_DOWN)
				{
					dx = 0;
					dy = 1;
					myfish.setLocation(myfish.getX() + dx, myfish.getY() + dy);
				}
			}
		}

		class fishmovethread extends Thread
		{
			public void run()
			{
				while (true)
				{
					try
					{
						Thread.sleep(10);
						fishmove();
						eating();
					} catch (InterruptedException e)
					{

					}
				}
			}
		}

		class enemyfishcrowd extends JLabel
		{
			boolean enemyalive = true; // 살아있으면 1
		}

	}

	public static void main(String arg[])
	{
		new growfishgame();
	}
}
