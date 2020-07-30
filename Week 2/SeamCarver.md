```java
package week2;

import edu.princeton.cs.algs4.Picture;

public class SeamCarver {
    private int[][] rgb; // store rgb value
    private double[][] dist; // used in findSeam
    private int[][] edgeTo; // used in findSeam
    private final int XR = 0xff << 16; // used in energy()
    private final int XG = 0xff << 8;
    private final int XB = 0xff;
    private int height;
    private int width;

    // create a seam carver object based on the given picture
    public SeamCarver(Picture picture) {
        if (picture == null) {
            throw new IllegalArgumentException();
        }
        height = picture.height();
        width = picture.width();
        rgb = new int[width][height];
        dist = new double[width][height];
        edgeTo = new int[width][height];
        int value;
        for (int col = 0; col < width; ++col) {
            for (int row = 0; row < height; ++row) {
                value = picture.getRGB(col, row);
                rgb[col][row] = value;
            }
        }
    }

    // current picture
    public Picture picture() {
        Picture res = new Picture(width, height);
        for (int col = 0; col <width; ++col) {
            for (int row = 0; row < height; ++row) {
                res.setRGB(col, row, rgb[col][row]);
            }
        }
        return res;
    }

    // width of current picture
    public int width() {
        return width;
    }

    // height of current picture
    public int height() {
        return height;
    }

    // energy of pixel at column x and row y
    public double energy(int x, int y) {
        if (!validateCoord(x, y)) {
            throw new IllegalArgumentException();
        }
        if (x == 0 || x == width() - 1 || y == 0 || y == height() - 1) { // if it's border, energy is 1000
            return 1000;
        }
        int dx2 = 0, dy2 = 0;
        int dr = getR(x + 1, y) - getR(x - 1, y);
        int dg = getG(x + 1, y) - getG(x - 1, y);
        int db = getB(x + 1, y) - getB(x - 1, y);
        dx2 = dr*dr + dg*dg + db*db;
        dr = getR(x, y + 1) - getR(x, y - 1);
        dg = getG(x, y + 1) - getG(x, y - 1);
        db = getB(x, y + 1) - getB(x, y - 1);
        dy2 = dr*dr + dg*dg + db*db;
        return Math.sqrt(dx2 + dy2);
    }

    private int getR(int x, int y) {
        return (rgb[x][y] & XR) >> 16;
    }

    private int getG(int x, int y) {
        return (rgb[x][y] & XG) >> 8;
    }

    private int getB(int x, int y) {
        return rgb[x][y] & XB;
    }

    // sequence of indices for horizontal seam
    public int[] findHorizontalSeam() {
        int[] seam = new int[width()];
        int maxDist = 1000;
        for (int c = 0; c < width(); ++c) {
            for (int r = 0; r < height(); ++r) {
                dist[c][r] = maxDist;
            }
        }
        for (int col = 1; col < width(); ++col) {
            for (int row = 0; row < height(); ++row) {
                int minRow = -1;
                for (int i = -1; i <= 1; ++i) {
                    if (validateCoord(col - 1, row + i)) {
                        if (minRow == -1) {
                            minRow = row + i;
                        }
                        else if (dist[col - 1][row + i] < dist[col - 1][minRow]) {
                            minRow = row + i;
                        }
                    }
                }
                edgeTo[col][row] = minRow;
                dist[col][row] = dist[col - 1][minRow] + energy(col, row);
            }
        }
        int minRow = 0;
        for (int row = 0; row < height(); ++row) {
            if (dist[width() - 1][row] < dist[width() - 1][minRow]) {
                minRow = row;
                seam[width() - 1] = minRow;
            }
        }
        for (int col = width() - 2; col >= 0; --col) {
            seam[col] = edgeTo[col + 1][seam[col + 1]];
        }
        return seam;
    }

    // sequence of indices for vertical seam
    public int[] findVerticalSeam() {
        int[] seam = new int[height()];
        int maxDist = 1000;
        for (int c = 0; c < width(); ++c) {
            for (int r = 0; r < height(); ++r) {
                dist[c][r] = maxDist;
            }
        }
        for (int row = 1; row < height(); ++row) {
            for (int col = 0; col < width(); ++col) {
                int minCol = -1;
                for (int i = -1; i <= 1; ++i) {
                    if (validateCoord(col + i, row - 1)) {
                        if (minCol == -1) {
                            minCol = col + i;
                        }
                        else if (dist[col + i][row - 1] < dist[minCol][row - 1]) {
                            minCol = col + i;
                        }
                    }
                }
                edgeTo[col][row] = minCol;
                dist[col][row] = dist[minCol][row - 1] + energy(col, row);
            }
        }
        int minCol = 0;
        for (int col = 0; col < width(); ++col) {
            if (dist[col][height() - 1] < dist[minCol][height() - 1]) {
                minCol = col;
                seam[height() - 1] = minCol;
            }
        }
        for (int row = height() - 2; row >= 0; --row) {
            seam[row] = edgeTo[seam[row + 1]][row + 1];
        }
        return seam;
    }

    // remove horizontal seam from current picture
    public void removeHorizontalSeam(int[] seam) {
        if (seam == null || seam.length != width() || height() <= 1) {
            throw new IllegalArgumentException();
        }
        for (int i = 0; i < seam.length; ++i) {
            if (!validateCoord(0, seam[i])) {
                throw new IllegalArgumentException();
            }
            if (i > 0 && Math.abs(seam[i] - seam[i - 1]) > 1) {
                throw new IllegalArgumentException();
            }
        }
        for (int col = 0; col < width(); ++col) {
            for (int row = seam[col]; row < height() - 1; ++row) {
                rgb[col][row] = rgb[col][row + 1];
            }
        }
        height -= 1;
    }

    // remove vertical seam from current picture
    public void removeVerticalSeam(int[] seam) {
        if (seam == null || seam.length != height() || width() <= 1) {
            throw new IllegalArgumentException();
        }
        for (int i = 0; i < seam.length; ++i) {
            if (!validateCoord(seam[i], 0)) {
                throw new IllegalArgumentException();
            }
            if (i > 0 && Math.abs(seam[i] - seam[i - 1]) > 1) {
                throw new IllegalArgumentException();
            }
        }
        for (int row = 0; row < height(); ++row) {
            for (int col = seam[row]; col < width() - 1; ++col) {
                rgb[col][row] = rgb[col + 1][row];
            }
        }
        width -= 1;
    }
    
    private boolean validateCoord(int x, int y) {
        if (x < 0 || x >= width() || y < 0 || y >= height()) {
            return false;
        }
        return true;
    }

    //  unit testing (optional)
    public static void main(String[] args) {
        // Picture p = new Picture("6m5.png");
        // SeamCarver sc = new SeamCarver(p);
        // for (int r = 0; r < sc.height(); ++r) {
        //     for (int c = 0; c < sc.width(); ++c) {
        //         System.out.printf("%10d", sc.rgb[c][r]);
        //     }
        //     System.out.println();
        // }
        // int[] a = sc.findVerticalSeam();
        // for (int i = 0; i < a.length; ++i) {
        //     System.out.print(" " + a[i]);
        // }
        // System.out.println();
        // sc.removeVerticalSeam(a);
        // for (int r = 0; r < sc.height(); ++r) {
        //     for (int c = 0; c < sc.width(); ++c) {
        //         System.out.printf("%10d", sc.rgb[c][r]);
        //     }
        //     System.out.println();
        // }
    }
}
```
