//# java-final-project
//java 언어를 이용하여 window 그림판 만들기
//package project;
//import javax.swing.*;
//import java.awt.*;
//import java.awt.event.*;
//import java.awt.image.BufferedImage;
//import java.io.File;
//import javax.imageio.ImageIO;
//import java.io.IOException;

//public class SimplePaint extends JFrame {
    //private BufferedImage canvas;
    //private Graphics2D g2d;
   // private int startX, startY, endX, endY;
    //private String currentTool = "Pencil";
    //private Color currentColor = Color.BLACK;
    //private boolean eraserEnabled = false;

    public SimplePaint() {
        setTitle("Simple Paint");
        setSize(800, 600);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        canvas = new BufferedImage(800, 600, BufferedImage.TYPE_INT_ARGB);
        g2d = canvas.createGraphics();
        g2d.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
        g2d.setPaint(Color.WHITE);
        g2d.fillRect(0, 0, canvas.getWidth(), canvas.getHeight());
        g2d.setPaint(Color.BLACK);

        JPanel toolsPanel = new JPanel();
        String[] tools = {"Pencil", "Line", "Rectangle", "Circle", "Point"};
        JComboBox<String> toolBox = new JComboBox<>(tools);
        toolBox.addActionListener(e -> {
            currentTool = (String) toolBox.getSelectedItem();
            eraserEnabled = false; // Disable eraser when another tool is selected
        });
        toolsPanel.add(toolBox);

        JButton colorButton = new JButton("Color");
        colorButton.addActionListener(e -> {
            Color selectedColor = JColorChooser.showDialog(null, "Choose a color", currentColor);
            if (selectedColor != null) {
                currentColor = selectedColor;
                g2d.setPaint(currentColor);
            }
        });
        toolsPanel.add(colorButton);

        JButton eraserButton = new JButton("Eraser");
        eraserButton.addActionListener(e -> eraserEnabled = !eraserEnabled);
        toolsPanel.add(eraserButton);

        JButton saveButton = new JButton("Save");
        saveButton.addActionListener(e -> saveImage());
        toolsPanel.add(saveButton);

        JButton loadButton = new JButton("Load");
        loadButton.addActionListener(e -> loadImage());
        toolsPanel.add(loadButton);

        add(toolsPanel, BorderLayout.NORTH);

        JPanel drawPanel = new JPanel() {
            @Override
            protected void paintComponent(Graphics g) {
                super.paintComponent(g);
                g.drawImage(canvas, 0, 0, null);
            }
        };
        drawPanel.setPreferredSize(new Dimension(800, 600));
        drawPanel.addMouseListener(new MouseAdapter() {
            @Override
            public void mousePressed(MouseEvent e) {
                startX = e.getX();
                startY = e.getY();
                if (currentTool.equals("Point") && !eraserEnabled) {
                    g2d.fillOval(startX, startY, 5, 5);
                    drawPanel.repaint();
                } else if (eraserEnabled) {
                    erase(startX, startY);
                    drawPanel.repaint();
                }
            }

            @Override
            public void mouseReleased(MouseEvent e) {
                endX = e.getX();
                endY = e.getY();
                drawShape();
                drawPanel.repaint();
            }
        });
        drawPanel.addMouseMotionListener(new MouseAdapter() {
            @Override
            public void mouseDragged(MouseEvent e) {
                if (currentTool.equals("Pencil") && !eraserEnabled) {
                    int x = e.getX();
                    int y = e.getY();
                    g2d.drawLine(startX, startY, x, y);
                    startX = x;
                    startY = y;
                    drawPanel.repaint();
                } else if (eraserEnabled) {
                    int x = e.getX();
                    int y = e.getY();
                    erase(x, y);
                    drawPanel.repaint();
                }
            }
        });
        add(new JScrollPane(drawPanel), BorderLayout.CENTER);
    }

    private void drawShape() {
        if (eraserEnabled) return;
        switch (currentTool) {
            case "Line":
                g2d.drawLine(startX, startY, endX, endY);
                break;
            case "Rectangle":
                g2d.drawRect(Math.min(startX, endX), Math.min(startY, endY),
                        Math.abs(startX - endX), Math.abs(startY - endY));
                break;
            case "Circle":
                int radius = (int) Math.sqrt(Math.pow(endX - startX, 2) + Math.pow(endY - startY, 2));
                g2d.drawOval(startX - radius, startY - radius, 2 * radius, 2 * radius);
                break;
            default:
                break;
        }
    }

    private void erase(int x, int y) {
        g2d.setPaint(Color.WHITE);
        g2d.fillRect(x - 10, y - 10, 20, 20);
        g2d.setPaint(currentColor);
    }

    private void saveImage() {
        JFileChooser fileChooser = new JFileChooser();
        fileChooser.setDialogTitle("Save Image");
        int userSelection = fileChooser.showSaveDialog(this);
        if (userSelection == JFileChooser.APPROVE_OPTION) {
            File fileToSave = fileChooser.getSelectedFile();
            try {
                // Ensure file has .png extension
                if (!fileToSave.getName().toLowerCase().endsWith(".png")) {
                    fileToSave = new File(fileToSave.getAbsolutePath() + ".png");
                }
                ImageIO.write(canvas, "png", fileToSave);
            } catch (IOException ex) {
                ex.printStackTrace();
            }
        }
    }

    private void loadImage() {
        JFileChooser fileChooser = new JFileChooser();
        fileChooser.setDialogTitle("Open Image");
        int userSelection = fileChooser.showOpenDialog(this);
        if (userSelection == JFileChooser.APPROVE_OPTION) {
            File fileToOpen = fileChooser.getSelectedFile();
            try {
                BufferedImage loadedImage = ImageIO.read(fileToOpen);
                g2d.drawImage(loadedImage, 0, 0, null);
                repaint();
            } catch (IOException ex) {
                ex.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            SimplePaint sp = new SimplePaint();
            sp.setVisible(true);
        });
    }
}

