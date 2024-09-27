---
title: "opencv-cpp"
date: 2024-03-23 08:48:27
categories: "图像处理"
tags: ["Cpp", "图像处理"]
summary: "opencv的图像处理 c++ 版本"
---

1. 对像素点进行处理，保证rgb数值不超过范围
    
    ```cpp
    int stemp(int a)
    {
        if (a > 255)
            a = 255;
        if (a < 0)
            a = 0;
        return a;
    }
    ```
    
2. 读入图像和想要实现的操作，实现相应的操作，例如灰度图，HSV图，LAC图等
    
    ```cpp
    void operate(cv::Mat image, const std::string operate, int light = 0, float contrast = 1)
    {
        cv::Mat dst;
        if (operate == "LAC")
        {
            int i, j;
            multiply(image, cv::Scalar(contrast, contrast, contrast), dst);
            dst = dst + cv::Scalar(light, light, light);
            for (i = 0; i < image.cols; i++)
            {
                for (j = 0; j < image.rows; j++)
                {
                    dst.at<cv::Vec3b>(i, j)[0] = stemp(dst.at<cv::Vec3b>(i, j)[0]);
                    dst.at<cv::Vec3b>(i, j)[1] = stemp(dst.at<cv::Vec3b>(i, j)[1]);
                    dst.at<cv::Vec3b>(i, j)[2] = stemp(dst.at<cv::Vec3b>(i, j)[2]);
                }
            }
            imshow("change", dst);
        }
        if (operate == "gray")
        {
            cvtColor(image, dst, cv::COLOR_GRAY2BGR);
            imshow("gray", dst);
        }
        if (operate == "HSV")
        {
            cvtColor(image, dst, cv::COLOR_HSV2BGR);
            imshow("HSV", dst);
        }
    }
    
    ```
    
3. 对图像亮度和对比度处理，主要是引用 `operate` 函数来实现，就是个 `createTrackBar` 的回调函数
    
    ```cpp
    int light = 50;
    int contrast = 50;
    void LightAndContrast(int, void *userdata)
    {
        cv::Mat image = *((cv::Mat *)userdata);
        float i = (float)contrast / 50;
        operate(image, "LAC", 2 * (light - 50), i);
    }
    ```
    
4. 滚动条操作 创建一个关于亮度和对比度的滚动条
    
    ```cpp
    void tracking_bar(cv::Mat &image)
    {
        cv::namedWindow("change", cv::WINDOW_AUTOSIZE);
        cv::imshow("change", image);
        int max = 100;
        cv::createTrackbar("light", "change", &light, max, LightAndContrast, (void *)(&image));
        cv::createTrackbar("ocntrast", "change", &contrast, max, LightAndContrast, (void *)(&image));
        LightAndContrast(0, &image);
    }
    ```
    
5. 通道分离函数，主要是把图像的三个颜色通道分离
    
    ```cpp
    void channel_demo(cv::Mat &image)
    {
        std::vector<cv::Mat> mv;
        split(image, mv);
        cv::Mat dst;
        mv[0] = 0;
        mv[1] = 0;
        merge(mv, dst); // 图像和
        int from_to[] = {0, 1, 1, 2, 2, 0};
        mixChannels(&image, 1, &dst, 1, from_to, 3);
        imshow("change", dst);
    }
    ```
    
6. 在图像上进行操作把范围内的颜色阈值给设为白色，其他的设为黑色
    
    ```cpp
    void inrange_demo(cv::Mat &image)
    {
        cv::Mat HSV;
        cvtColor(image, HSV, cv::COLOR_RGB2HSV);
        cv::Mat mask;
        inRange(HSV, cv::Scalar(240, 240, 240), cv::Scalar(255, 255, 255), mask);
        imshow("mask", mask);
    }
    ```
    
7. 像素值统计，对图像的3个颜色通道上的r|g|b的最大值和最小值提取，并且找到所在位置。计算三个通道的均值和标准差
    
    ```cpp
    void pixel_statistic_demo(cv::Mat &image)
    {
        std::vector<cv::Mat> mv;
        split(image, mv);
        double minv, maxv;
        cv::Point minLoc, maxLoc;
        for (int i = 0; i < 3; i++)
        {
            minMaxLoc(mv[i],      // mv[]是表示单个通道的图像，可以是灰度图
                      &minv,      // minv是通道的最小值
                      &maxv,      // 通道的最大值
                      &minLoc,    // 最小值的地址
                      &maxLoc,    // 最大值的地址
                      cv::Mat()); // 图像
            std::cout << "第" << i << "个通道" << std::endl;
            std::cout << "Max value" << maxv << std::endl;
            std::cout << "Min value" << minv << std::endl;
            std::cout << "Max location" << maxLoc << std::endl;
            std::cout << "Min Location" << minLoc << std::endl;
        }
        cv::Mat mean, stddev;
        meanStdDev(image, mean, stddev); //
        std::cout << "means" << mean << "\nstddev:" << stddev << std::endl;
    }
    ```
    
8. 在图像上绘制
    
    ```cpp
    void draw_demo(cv::Mat &image)
    {
        cv::Rect rect; //绘制矩形
        rect.x = 200;  //矩形开始的位置，左上角，屏幕左上角为0，0
        rect.y = 200;
        rect.width = 100; // 矩形的高度和宽度
        rect.height = 100;
        cv::Mat bg = cv::Mat::zeros(image.size(), image.type());
        cv::rectangle(bg, rect, cv::Scalar(0, 0, 255), -1, 8, 0);
        cv::circle(bg, cv::Point(100, 100), 15, cv::Scalar(255, 0, 0), -1, 8, 0);
        cv::line(bg, cv::Point(100, 100), cv::Point(200, 200), cv::Scalar(0, 255, 0), 2, cv::LINE_AA, 0); // LINE_AA 反锯齿
        cv::RotatedRect rrt;
        rrt.center = cv::Point(150, 150);
        rrt.size = cv::Size(100, 200);
        rrt.angle = 90.0;
        cv::ellipse(bg, rrt, cv::Scalar(0, 255, 255), 2, 8);
        cv::Mat dst;
        cv::addWeighted(image, 0.7, bg, 0.3, 0, dst); // 两张图片按照权重比叠加
        cv::imshow("deal", bg);
    }
    ```
    
9. 随机绘制直线
    
    ```cpp
    void random_draw(cv::Mat &image)
    {
        cv::Mat canvas = cv::Mat::zeros(image.size(), image.type());
        int w = image.cols;
        int h = image.rows;
        cv::RNG RNG(12345);
        while (true)
        {
            int c = cv::waitKey(10);
            if (c == 27)
                break;
            int x1 = RNG.uniform(0, w);
            int y1 = RNG.uniform(0, h);
            int x2 = RNG.uniform(0, w);
            int y2 = RNG.uniform(0, h);
            // canvas = cv::Scalar(0, 0, 0);   // 每次只显示一条线
            cv::line(canvas, cv::Point(x1, y1), cv::Point(x2, y2), cv::Scalar(RNG.uniform(0, 255), RNG.uniform(0, 255), RNG.uniform(0, 255)), 1, cv::LINE_AA, 0);
            imshow("random_line", canvas);
        }
    }
    ```
    
10. 绘制多边形，并且填充
    
    ```cpp
    void polying_draw(void)
    {
        cv::Mat canvas = cv::Mat::zeros(cv::Size(512, 512), CV_8UC3);
        int w = 512;
        int h = 512;
        cv::RNG rng(12345);
        cv::Point p1(rng.uniform(0, w), rng.uniform(0, h));
        cv::Point p2(rng.uniform(0, w), rng.uniform(0, h));
        cv::Point p3(rng.uniform(0, w), rng.uniform(0, h));
        cv::Point p4(rng.uniform(0, w), rng.uniform(0, h));
        cv::Point p5(rng.uniform(0, w), rng.uniform(0, h));
        std::vector<cv::Point> pts;
        pts.push_back(p1);
        pts.push_back(p2);
        pts.push_back(p3);
        pts.push_back(p4);
        pts.push_back(p5);
        // polylines(canvas, pts, true, cv::Scalar(0, 0, 255), 1, cv::LINE_AA, 0);
        // cv::fillPoly(canvas, pts, cv::Scalar(255, 255, 0), 8, 0);
        std::vector<std::vector<cv::Point>> contours;
        contours.push_back(pts);
        cv::drawContours(canvas, contours, -1, cv::Scalar(0, 255, 255), -1);
        imshow("draw", canvas);
    }
    ```
    
11. 图像的像素逻辑操作
    
    ```cpp
    void bitwise_demo(void)
    {
        cv::Mat m2 = cv::Mat::zeros(cv::Size(512, 512), CV_8UC3);
        cv::Mat m1 = cv::Mat::zeros(cv::Size(512, 512), CV_8UC3);
        cv::Mat dst_and;
        cv::Mat dst_or;
        cv::Mat dst_not;
        cv::rectangle(m1, cv::Rect(100, 100, 80, 80), cv::Scalar(0, 255, 255), -1, cv::LINE_8, 0);
        cv::rectangle(m2, cv::Rect(150, 150, 80, 80), cv::Scalar(255, 255, 0), -1, cv::LINE_8, 0);
        imshow("m1", m1);
        imshow("m2", m2);
        cv::bitwise_and(m1, m2, dst_and);
        imshow("and", dst_and);
        cv::bitwise_or(m1, m2, dst_or);
        imshow("or", dst_or);
        cv::bitwise_not(m1, dst_not);
        imshow("not", dst_not);
    }
    ```
    
12. 鼠标操作，在图像上绘制矩形，并且把矩形内的图像展示出来
    
    ```cpp
    cv::Point sp(-1, -1);
    cv::Point ep(-1, -1);
    cv::Mat temp;
    static void on_draw(int event, int x, int y, int flags, void *userdata)
    {
        cv::Mat image = *((cv::Mat *)userdata);
        if (event == cv::EVENT_LBUTTONDOWN)
        {
            sp.x = x;
            sp.y = y;
            std::cout << "start point" << sp << std::endl;
        }
        else if (event == cv::EVENT_LBUTTONUP)
        {
            ep.x = x;
            ep.y = y;
            int dx = ep.x - sp.x;
            int dy = ep.y - sp.y;
            if (dx > 0 && dy > 0)
            {
                cv::rectangle(image, cv::Rect(sp.x, sp.y, dx, dy), cv::Scalar(255, 255, 0), 1, cv::LINE_8, 0);
                // temp = image.clone();
            }
            cv::imshow("mouse", image);
            sp.x = -1;
            sp.y = -1;
        }
        else if (event == cv::EVENT_MOUSEMOVE)
        {
            if (sp.x > 0 && sp.y > 0)
            {
                ep.x = x;
                ep.y = y;
                int dx = ep.x - sp.x;
                int dy = ep.y - sp.y;
                if (dx > 0 && dy > 0)
                {
                    temp.copyTo(image);
                    cv::rectangle(image, cv::Rect(sp.x, sp.y, dx, dy), cv::Scalar(255, 255, 0), 1, cv::LINE_8, 0);
                    cv::imshow("mouse", image);
                    temp.copyTo(image);
                    cv::imshow("ROI", image(cv::Rect(sp.x, sp.y, dx, dy)));
                }
            }
        }
    }
    ```
    
13. 图像归一化
    
    ```cpp
    void norm_demo(cv::Mat &image)
    {
        cv::Mat dst;
        image.convertTo(image, CV_32F);                     // 变为float类型
        std::cout << image.type() << std::endl;             // CV_8UC3 ：16
        cv::normalize(image, dst, 1.0, 0, cv::NORM_MINMAX); // 把图像用最大最小值来归一化
        std::cout << dst.type() << std::endl;               // CV_32F : 21
        imshow("normalize", dst);
    }
    ```
    
14. 图像缩放
    
    ```cpp
    void resize_demo(cv::Mat &image)
    {
        cv::Mat zoomin, zoomout;
        int h = image.rows;
        int w = image.cols;
        cv::resize(image, zoomin, cv::Size(w / 2, h / 2), 0, 0, cv::INTER_LINEAR);
        cv::resize(image, zoomout, cv::Size(w * 2, h * 2), 0, 0, cv::INTER_LINEAR);
        cv::imshow("min", zoomout);
        cv::imshow("mout", zoomout);
    }
    
    ```
    
15. 图像的翻转
    
    ```cpp
    void flip_demo(cv::Mat &image)
    {
        cv::Mat dst;
        // cv::flip(image, dst, 0); // 上下翻转
        // cv::flip(image, dst, 1); // 左右翻转
        cv::flip(image, dst, -1); // 对角线翻转
        cv::imshow("0", dst);
    }
    ```
    
16. 图像的插值
    
    ```cpp
    void insert_demo(cv::Mat &image) {
    }
    ```
    
17. 图像的旋转
    
    ```cpp
    void rotate_demo(cv::Mat &image)
    {
        cv::Mat dst, M;
        int h = image.rows;
        int w = image.cols;
        M = cv::getRotationMatrix2D(cv::Point2f(w / 2, h / 2), 60, 1.0); // 根据旋转角度获得旋转矩阵
        double cos = abs(M.at<double>(0, 0));
        double sin = abs(M.at<double>(0, 1));
        int neww = cos * w + sin * h;
        int newh = sin * w + cos * h;
        M.at<double>(0, 2) = M.at<double>(0, 2) + (neww / 2 - w / 2);
        M.at<double>(1, 2) = M.at<double>(1, 2) + (newh / 2 - h / 2);
        cv::warpAffine(image, dst, M, cv::Size(neww, newh), cv::INTER_LINEAR, 0, cv::Scalar(255, 255, 0)); // 旋转图像，并且决定旋转后图像的底色
        imshow("rotation", dst);
    }
    ```
    
18. 视频读取和处理
    
    ```cpp
    void video_demo(void)
    {
        cv::VideoCapture capture("../image/empty.mp4");            // 捕获视频文件，0表示读取摄像头数据，可以是视频文件地址
        int frame_width = capture.get(cv::CAP_PROP_FRAME_WIDTH);   // 视频的宽度
        int frame_height = capture.get(cv::CAP_PROP_FRAME_HEIGHT); // 视频的高度
        int count = capture.get(cv::CAP_PROP_FRAME_COUNT);         // 视频的帧数
        double fps = capture.get(cv::CAP_PROP_FPS);                // 视频的fps
        std::cout << "width     " << frame_width << std::endl;
        std::cout << "height     " << frame_height << std::endl;
        std::cout << "count     " << count << std::endl; // 读取摄像头数据时，count为-1
        std::cout << "fps     " << fps << std::endl;
        cv::Mat frame;
        while (true)
        {
            capture.read(frame); // 将视频文件的图片读入frame中，可以对每一帧视频处理
            if (frame.empty())
            {
                break;
            }
            imshow("frame", frame);
            operate(frame, "HSV", 0, 0);
            int c = cv::waitKey(10);
            if (c == 27)
                break;
        }
        cv::VideoWriter writer("../image/1.mp4", capture.get(cv::CAP_PROP_FOURCC), fps, cv::Size(frame_width, frame_height), true);
        capture.release();
        writer.release();
    }
    ```
    
19. 三通道直方图，三个通道分别的直方图
    
    ```cpp
    void showHistogram(cv::Mat &image)
    {
        // 三通道分离
        std::vector<cv::Mat> bgr_plane;
        cv::split(image, bgr_plane);
        // 定义参数变量
        const int channels[1] = {0};
        const int bins[1] = {256};
        float hranges[2] = {0, 255};
        const float *ranges[1] = {hranges};
        cv::Mat b_hist; // 结果
        cv::Mat g_hist;
        cv::Mat r_hist;
        // 计算机三个通道的直方图
        cv::calcHist(&bgr_plane[0], 1, 0, cv::Mat(), b_hist, 1, bins, ranges);
        cv::calcHist(&bgr_plane[1], 1, 0, cv::Mat(), g_hist, 1, bins, ranges);
        cv::calcHist(&bgr_plane[2], 1, 0, cv::Mat(), r_hist, 1, bins, ranges);
        int hist_w = 512;
        int hist_h = 400;
        int bin_w = cvRound((double)hist_w / bins[0]);
        cv::Mat histImage = cv::Mat::zeros(hist_h, hist_w, CV_8UC3);
        // 归一化直方图数据
        cv::normalize(b_hist, b_hist, 0, histImage.rows, cv::NORM_MINMAX, -1, cv::Mat());
        cv::normalize(g_hist, g_hist, 0, histImage.rows, cv::NORM_MINMAX, -1, cv::Mat());
        cv::normalize(r_hist, r_hist, 0, histImage.rows, cv::NORM_MINMAX, -1, cv::Mat());
        // 绘制直方图曲线
        for (int i = 1; i < bins[0]; i++)
        {
            line(histImage, cv::Point(bin_w * (i - 1), hist_h - cvRound(b_hist.at<float>(i - 1))),
                 cv::Point(bin_w * (i), hist_h - cvRound(b_hist.at<float>(i))), cv::Scalar(255, 0, 0), 2, cv::LINE_AA, 0);
            line(histImage, cv::Point(bin_w * (i - 1), hist_h - cvRound(g_hist.at<float>(i - 1))),
                 cv::Point(bin_w * (i), hist_h - cvRound(g_hist.at<float>(i))), cv::Scalar(0, 255, 0), 2, cv::LINE_AA, 0);
            line(histImage, cv::Point(bin_w * (i - 1), hist_h - cvRound(r_hist.at<float>(i - 1))),
                 cv::Point(bin_w * (i), hist_h - cvRound(r_hist.at<float>(i))), cv::Scalar(0, 0, 255), 2, cv::LINE_AA, 0);
        }
        cv::namedWindow("HISTOGRAM", cv::WINDOW_AUTOSIZE);
        cv::imshow("HISTOGRAM", histImage);
    }
    ```
    
20. 灰度图像的像素直方图
    
    ```cpp
    void show_gray_histogram(cv::Mat image)
    {
        const int channels[1] = {0};
        const int bins[1] = {256};
        float hranges[2] = {0, 255};
        const float *ranges[1] = {hranges};
        cv::Mat gray_hist; // 结果
        cv::calcHist(&image, 1, 0, cv::Mat(), gray_hist, 1, bins, ranges);
        int hist_w = 512;
        int hist_h = 400;
        int bin_w = cvRound((double)hist_w / bins[0]);
        cv::Mat grayImage = cv::Mat::zeros(hist_h, hist_w, CV_8UC3);
        cv::normalize(gray_hist, gray_hist, 0, grayImage.rows, cv::NORM_MINMAX, -1, cv::Mat());
        for (int i = 1; i < bins[0]; i++)
        {
            line(grayImage, cv::Point(bin_w * (i - 1), hist_h - cvRound(gray_hist.at<float>(i - 1))),
                 cv::Point(bin_w * (i), hist_h - cvRound(gray_hist.at<float>(i))), cv::Scalar(255, 255, 255), 2, cv::LINE_AA, 0);
        }
        cv::imshow("HISTOGRAM", grayImage);
    }
    ```
    
21. 图像的二维直方图
    
    ```cpp
    void show_2D_Histogram(cv::Mat &image)
    {
        cv::Mat hsv, hs_hist;
        cv::cvtColor(image, hsv, cv::COLOR_BGR2HSV);
        cv::imshow("HSV", hsv);
        int hbins = 30, sbins = 32;
        int hist_bins[] = {hbins, sbins};
        float h_range[] = {0, 180};
        float s_range[] = {0, 256};
        const float *hs_ranges[] = {h_range, s_range};
        int hs_channels[] = {0, 1};
        cv::calcHist(&hsv, 1, hs_channels, cv::Mat(), hs_hist, 2, hist_bins, hs_ranges, true, false);
        double maxVal = 0;
        cv::minMaxLoc(hs_hist, 0, &maxVal, 0, 0);
        int scale = 10;
        cv::Mat hist2d_image = cv::Mat::zeros(sbins * scale, hbins * scale, CV_8UC3);
        for (int h = 0; h < hbins; h++)
        {
            for (int s = 0; s < sbins; s++)
            {
                float binVal = hs_hist.at<float>(h, s);
                int intensity = cvRound(binVal * 255 / maxVal);
                cv::rectangle(hist2d_image, cv::Point(h * scale, s * scale),
                              cv::Point((h + 1) * scale - 1, (s + 1) * scale - 1), cv::Scalar::all(intensity), -1);
            }
        }
        imshow("H_S image", hist2d_image);
    }
    ```
    
22. 图像的均衡化，并且输出图像的直方图
    
    ```cpp
    void histogram_eq(cv::Mat &image)
    {
        cv::Mat gray, dst;
        cv::cvtColor(image, gray, cv::COLOR_BGR2GRAY);
        cv::equalizeHist(gray, dst);
        cv::imshow("gray", gray);
        cv::imshow("equ", dst);
        show_gray_histogram(gray);
        show_gray_histogram(dst);
    }
    ```
    
23. 图像卷积，均值滤波
    
    ```cpp
    void blur_demo(cv::Mat &image)
    {
        cv::Mat dst;
        cv::blur(image, dst, cv::Size(15, 15), cv::Point(-1, -1));
        cv::imshow("Blur", dst);
    }
    ```
    
24. 高斯模糊，高斯滤波
    
    ```cpp
    void Gaussian_Blur(cv::Mat &image)
    {
        cv::Mat dst;
        cv::GaussianBlur(image, dst, cv::Size(0, 0), 15);
        cv::imshow("GaussianBlur", dst);
    }
    ```
    
25. 高斯双边模糊，就是对图像的某些特点隐藏，某些特点不隐藏
    
    ```cpp
    void bifilter_demo(cv::Mat &image)
    {
        cv::Mat dst;
        cv::bilateralFilter(image, dst, 0, 100, 10);
        cv::imshow("bifilter", dst);
    }
    ```

26. 实现图片的自动打码

    ```cpp
    bool Mosaic(cv::Mat image, std::vector<cv::Rect> &area)
    {
        if (area.empty())
        {
            std::cout << "未读取到区域 " << std::endl;
            return false;
        }
        int step = 10;
        cv::Mat dst = image;
        for (int t = 0; t < area.size(); t++)
        {
            int x = area[t].tl().x;
            int y = area[t].tl().y;
            int width = area[t].width;
            int height = area[t].height;
            // std::cout << x << std::endl;
            // std::cout << y << std::endl;
            // std::cout << width << std::endl;
            // std::cout << height << std::endl;
            for (int i = y; i < y + height; i += step)
            {
                for (int j = x; j < x + width; j += step)
                {
                    for (int m = i; m < step + i; m++)
                    {
                        for (int n = j; n < step + j; n++)
                        {
                            for (int c = 0; c < 3; c++)
                            {
                                dst.at<cv::Vec3b>(m, n)[c] = image.at<cv::Vec3b>(i, j)[c];
                            }
                        }
                    }
                }
            }
        }
        // cv::imshow("MOSAIC", dst);
        return true;
    }
    ```

27. 马赛克的测试函数

    ```cpp
    void testMosaic(cv::Mat image)
    {
        std::vector<cv::Rect> face;
        face.push_back(cv::Rect(100, 100, 200, 200));
        // std::cout << "未读取到区域 " << std::endl;
        // face.push_back(cv::Rect(300, 300, 200, 200));
        Mosaic(image, face);
    }
    ```

28. 自动人脸目标检测

   ```cpp
   bool areademo(cv::Mat image)
    {
        std::string file = "../haarcascade_frontalface_default.xml";
        // 创建人脸检测
        cv::CascadeClassifier object;
        object.load(file);
        std::vector<cv::Rect> face;
        object.detectMultiScale(image, face, 1.1, 5);
        if (area_retangle(image, face))
            return false;
        // cv::imshow("face_mosaic", image);
        return true;
    }
    ```

29. 在指定的区域画框
 
    ```cpp
    bool area_retangle(cv::Mat image, std::vector<cv::Rect> &area)
    {
        if (area.empty())
        {
            std::cout << "未读取到区域 " << std::endl;
            return false;
        }
        for (int t = 0; t < area.size(); t++)
        {
            int x1 = area[t].tl().x;
            int y1 = area[t].tl().y;
            int width = area[t].width;
            int height = area[t].height;
            int x2 = x1 + width;
            int y2 = y1 + width;
            cv::rectangle(image, cv::Rect(x1, y1, width, height), cv::Scalar(255, 255, 0), 1, cv::LINE_8, 0);
        }
        cv::imshow("face", image);
        // while (cv::waitKey(0) != 27)
        // ;
        return true;
    }
    ```

30. 颜色风格

    ```cpp
    void colorMap_Demo(cv::Mat &image)
    {
        int colorMap[] = {
            cv::COLORMAP_AUTUMN,
            cv::COLORMAP_BONE,
            cv::COLORMAP_JET,
            cv::COLORMAP_WINTER,
            cv::COLORMAP_RAINBOW,
            cv::COLORMAP_OCEAN,
            cv::COLORMAP_SUMMER,
            cv::COLORMAP_SPRING,
            cv::COLORMAP_COOL,
            cv::COLORMAP_HSV,
            cv::COLORMAP_PINK,
            cv::COLORMAP_HOT,
            cv::COLORMAP_PARULA,
            cv::COLORMAP_MAGMA,
            cv::COLORMAP_INFERNO,
            cv::COLORMAP_PLASMA,
            cv::COLORMAP_VIRIDIS,
            cv::COLORMAP_CIVIDIS,
            cv::COLORMAP_TWILIGHT,
            cv::COLORMAP_TWILIGHT_SHIFTED,
            cv::COLORMAP_TURBO,
            cv::COLORMAP_DEEPGREEN};
        cv::Mat dst;
        int index = 0;
        while (true) {
            int c = cv::waitKey(500);
            if (c == 27)
                break;
            cv::applyColorMap(image, dst, colorMap[index % 21]);
            index++;
            imshow("style", dst);
        }
    }
    ```


