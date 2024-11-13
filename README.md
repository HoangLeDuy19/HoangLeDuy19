#include <iostream>
#include <fstream>
#include <string>
#include <freeglut.h>
#include <vector_matrix.h>
#include <cmath>

float birdY = 0.0;          // Vị trí Y của con chim
float birdSpeed = 0.01;     // Tốc độ di chuyển lên xuống
bool movingUp = true;       // Hướng bay lên
float wingAngle = 0.0;      // Góc quay của cánh
float wingSpeed = 5.0;      // Tốc độ vỗ cánh

void init() {
    glClearColor(0.5, 0.8, 1.0, 1.0);
    glEnable(GL_DEPTH_TEST);
}

void drawBody() {
    glColor3f(1.0, 0.8, 0.0);  // Màu vàng cam cho thân
    glPushMatrix();
    glScalef(0.5, 0.3, 0.3);   // Tỉ lệ thân 
    glutSolidSphere(0.25, 20, 20);  // Hình cầu làm thân
    glPopMatrix();
}

void drawHead() {
    glColor3f(1.0, 0.8, 0.0);
    glPushMatrix();
    glTranslatef(0.22, 0.2, 0.0);
    glutSolidSphere(0.1, 20, 15);  // Hình cầu làm đầu
    glPopMatrix();
}

void drawBeak() {
    glColor3f(1.0, 0.5, 0.0);
    glPushMatrix();
    glTranslatef(0.35, 0.2, 0.0);   // Vị trí mỏ
    glRotatef(-90, 0.0, 1.0, 0.0);
    glutSolidCone(0.02, 0.15, 10, 2);  // Hình nón làm mỏ
    glPopMatrix();
}

void drawWing(float x, float y, float z, float angle) {
    glColor3f(0.9, 0.7, 0.1);
    glPushMatrix();
    glTranslatef(x, y, z);      // Vị trí cánh
    glRotatef(angle, 0.0, 0.0, 1.0);
    glScalef(0.5, 0.1, 0.05);
    glutSolidSphere(0.25, 20, 20);
    glPopMatrix();
}

void drawTail() {
    glColor3f(1.0, 0.7, 0.1);
    glPushMatrix();
    glTranslatef(-0.15, -0.0, 0.0);
    glRotatef(-45, 0.0, 0.0, 1.0);
    glScalef(0.1, 0.3, 0.05);
    glutSolidCone(0.25, 0.35, 10, 2);
    glPopMatrix();
}

void drawEye(float x, float y, float z) {
    glColor3f(0.0, 0.0, 0.0);
    glPushMatrix();
    glTranslatef(x, y, z);      // Vị trí mắt
    glutSolidSphere(0.02, 10, 10);
    glPopMatrix();
}

// New function to draw a leg with claws
void drawLeg(float x, float y, float z) {
    glColor3f(0.6, 0.3, 0.0);  // Màu nâu cho chân
    glPushMatrix();
    glTranslatef(x, y, z);      // Vị trí chân
    glRotatef(-90, 1.0, 0.0, 0.0);  // Xoay chân xuống dưới

    // Draw leg
    GLUquadric* quad = gluNewQuadric();
    gluCylinder(quad, 0.01, 0.01, 0.1, 10, 10);  // Smaller cylinder for the leg
    gluDeleteQuadric(quad);

    glTranslatef(0.0, 0.0, 0.1);  // Move to the end of the leg

    // Draw foot sphere
    glutSolidSphere(0.02, 10, 10);

    // Draw claws
    for (int i = 0; i < 3; ++i) {
        glPushMatrix();
        float angle = i * 30.0 - 30.0; // Spread claws around the foot
        glRotatef(angle, 0.0, 1.0, 0.0);  // Rotate to position claws
        glTranslatef(0.0, 0.0, 0.03);     // Move claw forward
        glRotatef(-45, 1.0, 0.0, 0.0);    // Angle claw downwards
        glutSolidCone(0.005, 0.03, 10, 2); // Small cone for each claw
        glPopMatrix();
    }

    glPopMatrix();
}

void display() {
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    glLoadIdentity();
    gluLookAt(0.0, 0.5, 2.0, 0.0, 0.0, 0.0, 0.0, 1.0, 0.0);

    // Di chuyển con chim lên và xuống để tạo hiệu ứng bay
    glPushMatrix();
    glTranslatef(0.0, birdY, 0.0);

    drawBody();
    drawHead();
    drawBeak();
    drawWing(0.05, 0.05, 0.2, 30 + sin(wingAngle) * 15);
    drawWing(0.05, 0.05, -0.2, -30 - sin(wingAngle) * 15);
    drawTail();
    drawEye(0.28, 0.23, 0.05);
    drawEye(0.28, 0.23, -0.05);

    // Draw legs closer to the body center
    drawLeg(-0.05, -0.15, 0.05);  // Left leg
    drawLeg(-0.05, -0.15, -0.05); // Right leg

    glPopMatrix();
    glutSwapBuffers();
}

void update(int value) {
    if (movingUp) {
        birdY += birdSpeed;
        if (birdY > 0.1) movingUp = false;
    }
    else {
        birdY -= birdSpeed;
        if (birdY < -0.1) movingUp = true;
    }
    wingAngle += 0.1 * wingSpeed;

    glutPostRedisplay();
    glutTimerFunc(16, update, 0);
}

void reshape(int w, int h) {
    glViewport(0, 0, w, h);
    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();
    gluPerspective(45.0, (float)w / (float)h, 1.0, 10.0);
    glMatrixMode(GL_MODELVIEW);
}

int main(int argc, char** argv) {
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGB | GLUT_DEPTH);
    glutInitWindowSize(800, 600);
    glutCreateWindow("Flying Bird with Flapping Wings - OpenGL");

    init();
    glutDisplayFunc(display);
    glutReshapeFunc(reshape);
    glutTimerFunc(0, update, 0);
    glutMainLoop();
    return 0;
}
