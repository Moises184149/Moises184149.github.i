# Moises184149.github.i
Geometry Dash por camara

En base a videos tutoriales del canal https://www.youtube.com/playlist?list=PLydhOMppcoU4UYgEvH7WTeC_znQi_y-MX se realizo el codigo adecuandolo para geometry dash al este tener la funcion de al posicionar el objeto se presionaba la tecla una sola vez, asi que se agrego dos variables boleanas siendo upPressed y downPressed. Estas al activarse dejaran la tecla presionada ahora cuando se mueve le objeto las teclas estaran presionadas hasta que vuelva a su posicion incial el objeto.

    #include <opencv2/core.hpp>
    #include <opencv2/opencv.hpp>
    #include <iostream>//funciones, condiciones
    #include <vector>//posicion contorno y cordenaas del objeto
    #include <Windows.h>//control d ela camra y consola
    
    using namespace cv;
    using namespace std;
    
    void PressKey(int key) {
        INPUT input = { 0 };
        input.type = INPUT_KEYBOARD;
        input.ki.wVk = key;
        input.ki.dwFlags = 0; // key down
        SendInput(1, &input, sizeof(INPUT));
    }
    
    void ReleaseKey(int key) {
        INPUT input = { 0 };
        input.type = INPUT_KEYBOARD;
        input.ki.wVk = key;
        input.ki.dwFlags = KEYEVENTF_KEYUP; // key up
        SendInput(1, &input, sizeof(INPUT));
    }
    
    int main() {
        Mat frame, image_hsv, mask;
        VideoCapture cap(0);//camara web, en caso de ser una de video externa colocar el valor en 1
        double maxArea;
        vector<Point> largestContours;
        int lowerHue = 100, lowerSaturation = 110, lowerValue = 160;//establecer tonos que detecta la camra en este caso un azul marino
        int upperHue = 255, upperSaturation = 255, upperValue = 255;
        
        bool upPressed = false, downPressed = false;//Mantener presionada las teclas
    
        if (!cap.isOpened()) {//si no reconoce ninguna camara
            cout << "Error al leer la cámara " << endl;
            return 1;
        }
    
        namedWindow("Settings", WINDOW_AUTOSIZE);
    
        while (1) {
            cap >> frame;
    
            if (frame.empty()) {
                break;
            }
    
            cvtColor(frame, image_hsv, COLOR_BGR2HSV);
    
            Scalar lowerThreshold(lowerHue, lowerSaturation, lowerValue);
            Scalar upperThreshold(upperHue, upperSaturation, upperValue);
    
            inRange(image_hsv, lowerThreshold, upperThreshold, mask);
            vector<vector<Point>> contours;
            findContours(mask, contours, RETR_EXTERNAL, CHAIN_APPROX_SIMPLE);
    
            maxArea = 0.0;
            for (int i = 0; i < contours.size(); i++) {
                double area = contourArea(contours[i]);
                if (area > maxArea) {
                    maxArea = area;
                    largestContours = contours[i];
                }
            }
    
            if (!largestContours.empty()) {
                Moments m = moments(largestContours);
                int cx = int(m.m10 / m.m00);//coordenadas prederterminadas de x
                int cy = int(m.m01 / m.m00);//coordenadas prederterminadas de y
    
                vector<vector<Point>> contoursToDraw = { largestContours };
                drawContours(frame, contoursToDraw, -1, Scalar(255, 255, 255), 3);//contorno del objeto
                circle(frame, Point(cx, cy), 5, Scalar(255, 255, 255), -1);//centro del objeto
    
                if (cy < frame.rows / 3) {
                    if (!upPressed) {
                        PressKey(VK_UP);
                        upPressed = true;
                    }
                }
                else {
                    if (upPressed) {
                        ReleaseKey(VK_UP);
                        upPressed = false;
                    }
                }
    
                if (cy > 2 * frame.rows / 3) {
                    if (!downPressed) {
                        PressKey(VK_DOWN);
                        downPressed = true;
                    }
                }
                else {
                    if (downPressed) {
                        ReleaseKey(VK_DOWN);
                        downPressed = false;
                    }
                }
            }
            else {
                if (upPressed) {
                    ReleaseKey(VK_UP);
                    upPressed = false;
                }
                if (downPressed) {
                    ReleaseKey(VK_DOWN);
                    downPressed = false;
                }
            }
    
            imshow("Settings", mask);
            imshow("Original frame", frame);
    
            char c = (char)waitKey(25);
            if (c == 27) { // ESC
                break;
            }
        }
    
        cap.release();
        destroyAllWindows();
    
        return 0;
    }
