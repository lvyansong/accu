# 双线性算法

在计算机图形学中, 图像缩放指的是通过增加或去掉像素来改变图片的尺寸. 由于要在效率和图像质量比如平滑度和清晰度之间做折衷, 图像缩放并不是个平庸的过程. 当图像尺寸增大的时候, 组成图像的像素也越来越大, 图像看上去就变"柔和"了. 而缩小图像的时候, 图像就变得平滑和清晰了. 本文介绍**双线性算法(Bilinear algorithms)**.

# 原理
双线性插值法与[最近邻插值法](/content/pil/resize_nearst/)类似, 不同点是取原图像中距离目标像素点最近的 4 个点, 并对这 4 个点与其对应权值的乘积求和, 获得最终像素值.

如下图:

![img](/img/pil/resize_bilinear/bilinear_interpolation.jpg)

目标点为 $$P$$, 距离 $$P$$ 最近的四个点为 $$Q_11$$, $$Q_12$$, $$Q_21$$, $$Q_22$$, 与 $$P$$ 围成的面积为 $$S_11$$, $$S_12$$, $$S_21$$, $$S_22$$, 分别以黄, 红, 青, 橙标出. 由于 $$S_11 + S_12 + S_21 + S_22 = 1$$, 因此最终求得的 $$P$$ 的像素值为

$$
P = Q_11 * S_22 + Q_12*S_21 + Q_21*S_12 + Q_22*S_11
$$

# 代码实现

为了方便计算, 下述程序将图像转换为矩阵进行操作.

```py
import numpy as np
import PIL.Image
import scipy.misc

im = PIL.Image.open('/img/jp.jpg')
im_mat = scipy.misc.fromimage(im)
im_mat_resized = np.empty((270, 480, im_mat.shape[2]), dtype=np.uint8)

for r in range(im_mat_resized.shape[0]):
    for c in range(im_mat_resized.shape[1]):
        rr = (r + 1) / im_mat_resized.shape[0] * im_mat.shape[0] - 1
        cc = (c + 1) / im_mat_resized.shape[1] * im_mat.shape[1] - 1

        rr_int = int(rr)
        cc_int = int(cc)

        if rr == rr_int and cc == cc_int:
            p = im_mat[rr_int][cc_int]
        elif rr == rr_int:
            p = im_mat[rr_int][cc_int] * (cc_int + 1 - cc) + im_mat[rr_int][cc_int + 1] * (cc - cc_int)
        elif cc == cc_int:
            p = im_mat[rr_int][cc_int] * (rr_int + 1 - rr) + im_mat[rr_int + 1][cc_int] * (rr - rr_int)
        else:
            p11 = (rr_int, cc_int)
            p12 = (rr_int, cc_int + 1)
            p21 = (rr_int + 1, cc_int)
            p22 = (rr_int + 1, cc_int + 1)

            dr1 = rr - rr_int
            dr2 = rr_int + 1 - rr
            dc1 = cc - cc_int
            dc2 = cc_int + 1 - cc

            w11 = dr2 * dc2
            w12 = dr2 * dc1
            w21 = dr1 * dc2
            w22 = dr1 * dc1

            p = im_mat[p11[0]][p11[1]] * w11 + im_mat[p21[0]][p21[1]] * w12 + \
                im_mat[p12[0]][p12[1]] * w21 + im_mat[p22[0]][p22[1]] * w22

        im_mat_resized[r][c] = p


im_resized = PIL.Image.fromarray(im_mat_resized)
im_resized.show()
```

# 优化
详见 [https://en.wikipedia.org/wiki/Bilinear_interpolation](https://en.wikipedia.org/wiki/Bilinear_interpolation)
