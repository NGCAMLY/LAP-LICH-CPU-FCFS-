#include <stdio.h>
// Cau truc luu thong tin tien trinh
typedef struct {
    int pid;    // ID cua tien trinh
    int burst;  // Thoi gian xu ly (CPU time)
} Process;
// Ham sap xep cac tien trinh theo thoi gian xu ly tang dan
void sortProcessesByBurst(Process proc[], int n) {
    for (int i = 0; i < n - 1; i++) {
        for (int j = 0; j < n - i - 1; j++) {
            if (proc[j].burst > proc[j + 1].burst) {
                // Hoan doi hai tien trinh
                Process temp = proc[j];
                proc[j] = proc[j + 1];
                proc[j + 1] = temp;
            }
        }
    }
}
// Ham ve bieu do Gantt và ghi vao file
void drawGanttChart(Process proc[], int n, int start_time[], FILE *file) {
    fprintf(file, "\nBieu do Gantt:\n");
    // Dong tren cung
    for (int i = 0; i < n; i++){
  fprintf(file, " ");
        for (int j = 0; j < proc[i].burst; j++) fprintf(file, "--");
        fprintf(file, " ");
    }
    fprintf(file, "\n|");
    // In cac tien trinh
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < proc[i].burst - 1; j++) fprintf(file, " ");
        fprintf(file, "P%d", proc[i].pid);
        for (int j = 0; j < proc[i].burst - 1; j++) fprintf(file, " ");
        fprintf(file, "|");
    }
    fprintf(file, "\n");
    // Dong cuoi cung
    for (int i = 0; i < n; i++) {
        fprintf(file, " ");
        for (int j = 0; j < proc[i].burst; j++) fprintf(file, "--");
        fprintf(file, " ");
    }
    fprintf(file, "\n");
    // In thoi gian phia duoi
    fprintf(file, "0"); // Thoi gian bat dau tu 0
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < proc[i].burst; j++) fprintf(file, "  ");
        fprintf(file, "%d", start_time[i] + proc[i].burst);
    }
   fprintf(file, "\n");
    // Them thong tin ve tien trinh dang chay toi moi moc thoi gian
    fprintf(file, "\nTien trinh dang chay toi moi moc thoi gian:\n");
    for (int i = 0; i < n; i++) {
        fprintf(file, "Tu thoi gian %d den %d, tien trinh P%d dang chay.\n", start_time[i], start_time[i] + proc[i].burst, proc[i].pid);
    }
}
// Ham lap lich CPU su dung FCFS
void FCFS(Process proc[], int n, FILE *file) {
    int wait_time[n];        // Mang luu thoi gian cho cho tung tien trinh
    int turnaround_time[n];  // Mang luu thoi gian hoan thanh cho tung tien trinh
    int start_time[n];       // Mang luu thoi gian bat dau xu ly cua tung tien trinh
    int total_wait = 0, total_turnaround = 0; // Tung thoi gian cho va thoi gian hoan thanh

    // Tính thoi gian bat dau, thoi gian cho và thoi gian hoan thanh
    wait_time[0] = 0; // Tien trinh dau tien khong con cho
    start_time[0] = 0; // Tien trinh dau tien bat dau tu thoi gian 0
    turnaround_time[0] = proc[0].burst; // Thoi gian hoan thanh = thoi gian xu ly voi tien trinh dau tien.
    for (int i = 1; i < n; i++) {
        wait_time[i] = wait_time[i - 1] + proc[i - 1].burst; // Thoi gian cho cua tien trinh i
        start_time[i] = wait_time[i]; // Thoi gian bat dau cua tien trinh i
        turnaround_time[i] = wait_time[i] + proc[i].burst; // Thoi gian hoan thanh cua tien trinh i
    }

    // Tính tong thoii gian cho và thoii gian hoan thanh
    for (int i = 0; i < n; i++) {
        total_wait += wait_time[i];
        total_turnaround += turnaround_time[i];
    }

    // In ket qua
    fprintf(file, "\nID tien trinh\tThoi gian xu ly\tThoi gian cho\tThoi gian hoan thanh\tThoi gian bat dau\n");
    for (int i = 0; i < n; i++) {
        fprintf(file, "%d\t\t%d\t\t%d\t\t%d\t\t\t%d\n", proc[i].pid, proc[i].burst, wait_time[i], turnaround_time[i], start_time[i]);
    }

    // Ve bieu do Gantt
    drawGanttChart(proc, n, start_time, file);

    // In thoi gian trung binh
    fprintf(file, "\nThoi gian cho trung binh: %.2f\n", (float)total_wait / n);
    fprintf(file, "Thoi gian hoan thanh trung binh: %.2f\n", (float)total_turnaround / n);
}

int main() {
    Process proc[] = {{1, 5}, {2, 3}, {3, 8}};  // Vi du cac tien trinh
    int n = sizeof(proc) / sizeof(proc[0]);

    FILE *file = fopen("output.txt", "w");
    if (!file) {
        printf("Khong the mo file output.txt da ghi du lieu.\n");
        return 1;
    }

    fprintf(file, "Danh sach tien trinh ban dau:\n");
    for (int i = 0; i < n; i++) {
        fprintf(file, "ID: %d, Thoi gian xu ly: %d\n", proc[i].pid, proc[i].burst);
    }

    // Sap xep cac tien trinh theo thoi gian xu ly
    sortProcessesByBurst(proc, n);

    fprintf(file, "\nDanh sach tien trinh sau khi sap xep theo thoi gian xu ly:\n");
    for (int i = 0; i < n; i++) {
        fprintf(file, "ID: %d, Thoi gian xu ly: %d\n", proc[i].pid, proc[i].burst);
    }

    // Giai thuat toan FCFS sau khi sap xep
    fprintf(file, "\nKet qua lap lich FCFS:\n");
    FCFS(proc, n, file);

    fclose(file);
    printf("Ket qua da duoc ghi vào file output.txt.\n");

    return 0;
}
