# PROJECT CONTEXT: MIPS 32-bit 5-Stage Pipelined Processor

## 1. Project Overview
This project implements a 32-bit MIPS processor using a 5-stage pipelined architecture (Instruction Fetch, Instruction Decode, Execute, Memory Access, Write Back). The design is strictly at the Register Transfer Level (RTL) using Verilog. It includes hazard resolution mechanisms (Data Forwarding and Pipeline Stalling) to handle data and control hazards efficiently.

## 2. Tech Stack & Toolchain
- **Hardware Description Language (HDL):** Verilog (IEEE 1364-2005 standard). Do NOT use SystemVerilog specific syntax.
- **Simulation & Compilation:** Icarus Verilog (iVerilog).
- **Waveform Viewer:** GTKWave (requires `$dumpfile` and `$dumpvars` in testbenches).
- **Assembler:** MARS MIPS Simulator (used externally to generate `.hex` machine code files).

## 3. Directory Structure
The AI Agent must strictly create and maintain the following folder structure:
```text
mips-32bit-pipeline/
│
├── asm/                    # Nơi chứa code MIPS Assembly và mã máy
│   ├── fibonacci.s         # Code hợp ngữ test tính Fibonacci
│   └── instruction.hex     # Mã máy (Machine code) xuất ra từ MARS để nạp vào IMEM
│
├── src/                    # Chứa TOÀN BỘ source code Verilog của thiết kế (RTL)
│   ├── alu.v
│   ├── regfile.v
│   ├── control_unit.v
│   ├── data_memory.v
│   ├── inst_memory.v
│   ├── hazard_unit.v
│   ├── forwarding_unit.v
│   ├── pipeline_regs.v     # Chứa các thanh ghi IF/ID, ID/EX, EX/MEM, MEM/WB
│   └── mips_top.v          # File Top-level kết nối toàn bộ hệ thống
│
├── tb/                     # Chứa code Testbench (Môi trường kiểm thử)
│   └── mips_tb.v           # File testbench cấp clk, reset và đọc file .hex
│
├── scripts/                # Script tự động hóa
│   └── run_sim.bat         # (Hoặc run_sim.sh) Lệnh chạy iVerilog và mở GTKWave tự động
│
├── waves/                  # Lưu các file config cấu hình dạng sóng của GTKWave
│   └── debug_layout.gtkw
│
├── docs/                   # Nơi chứa báo cáo và sơ đồ
│   └── report_MI4344.pdf
│
├── .gitignore              # Bỏ qua các file rác (.vcd, file thực thi .vvp)
└── README.md               # Hướng dẫn setup cho người mới
4. Hardware Architecture Specifications
Data Width: 32-bit data and 32-bit instruction width.

Memory Architecture: Harvard Architecture (Separate Instruction Memory and Data Memory).

Registers: 32 general-purpose 32-bit registers. Register $0 is hardwired to 0.

Pipeline Stages:

IF (Instruction Fetch): PC update, Instruction Memory read.

ID (Instruction Decode): Register read, Sign extension, Main Control Unit, Hazard Detection (Stall).

EX (Execute): ALU operations, ALU Control, Branch address calculation, Forwarding multiplexing.

MEM (Memory): Data Memory read/write, Branch decision.

WB (Write Back): Write data to Register File.

Supported Instructions (Subset of MIPS ISA):

R-type: add, sub, and, or, slt.

I-type: lw, sw, addi, beq.

J-type: j (Jump).

5. Module Breakdown & File Naming Convention (Strict)
The AI Agent must generate the following files inside the src/ directory with exact names:

5.1. Datapath & Processing Modules
alu.v: 32-bit ALU supporting ADD, SUB, AND, OR, SLT. Outputs Result and Zero flag.

regfile.v: 32x32-bit Register File. Two read ports (asynchronous), one write port (synchronous on falling edge of clock to resolve half-cycle data hazards).

sign_extend.v: Extends 16-bit immediate to 32-bit.

5.2. Memory Modules
inst_memory.v: Read-only memory. Must use $readmemh to load instruction.hex from the asm/ folder during initialization. Word-addressable (PC/4).

data_memory.v: Read/Write RAM. Synchronous write, asynchronous read.

5.3. Control Modules
control_unit.v: Decodes 6-bit Opcode to generate signals: RegDst, Jump, Branch, MemRead, MemtoReg, ALUOp, MemWrite, ALUSrc, RegWrite.

alu_control.v: Decodes 2-bit ALUOp and 6-bit funct field into 4-bit ALU control signal.

5.4. Pipeline Registers
pipeline_regs.v: Contains 4 sub-modules (or one large module with blocks) for IF_ID, ID_EX, EX_MEM, and MEM_WB registers. Must handle asynchronous reset and synchronous enable (for stalling) and synchronous clear (for flushing).

5.5. Hazard Units
forwarding_unit.v: Solves EX/MEM and MEM/WB data hazards by controlling MUXes before the ALU.

hazard_unit.v: Solves Load-Use data hazards by stalling the PC and IF/ID registers, and flushing ID/EX control signals to 0.

5.6. Top-Level Integration
mips_top.v: The top-level wrapper that instantiates all the above modules and connects them using wires. Represents the complete Datapath + Control.

6. Coding Guidelines & Constraints for AI Agent
Clock & Reset: Use active-high reset. Flip-flops should trigger on posedge clk or posedge reset, EXCEPT the Register File write operation which should ideally happen on negedge clk to ensure data written in WB is readable in the ID stage of the same cycle.

Testbench Requirement: Any testbench generated in tb/ (e.g., mips_tb.v) MUST include the following block for iVerilog/GTKWave compatibility:
initial begin
    $dumpfile("mips_sim.vcd");
    $dumpvars(0, testbench_module_name);
end

3. **No Blackboxes:** Do not use pre-compiled IPs. Everything must be purely synthesized behavioral or structural Verilog.
4. **Commenting:** Add brief, meaningful comments explaining the inputs, outputs, and internal logic (especially for hazard resolution). Use English for comments.
5. **Progressive Generation:** When asked to start, generate files step-by-step (Sprint by Sprint) rather than outputting all files at once to avoid context overflow.

Sau đây là context dự án bằng tiếng việt, nhóm tôi có 4 thành viên 
Đề tài: Thiết kế và Mô phỏng bộ vi xử lý MIPS 32-bit cấu trúc Pipelined 5 tầng 

TỔNG QUAN
Yêu cầu mọi người cài đặt 4 công cụ sau:
Ngôn ngữ thiết kế: Verilog (Tiêu chuẩn IEEE 1364-2005) 
Trình biên dịch & Mô phỏng: Icarus Verilog (iVerilog) 
Trình xem dạng sóng: GTKWave (Dùng để debug tín hiệu). 
Trình dịch Assembly: MARS MIPS Simulator chúng ta vẫn dùng lâu nay. Dùng để viết code MIPS Assembly (ví dụ tính giai thừa) và xuất ra file .hex nạp vào bộ nhớ lệnh. 
IDE/Editor: VS Code cài sẵn extension Verilog-HDL/SystemVerilog/Bluespec SystemVerilog (của mshr-h) để format code và check lỗi syntax. 
Quản lý mã nguồn: Git & GitHub. Yêu cầu code push lên repo chung tạo một branch riêng để anh check trước khi merge vào main branch, không gửi file qua Zalo/Messenger.

PHÂN CHIA FILE & NHIỆM VỤ CHI TIẾT
Dựa vào cấu trúc dự án nêu trên, anh chốt phân việc như sau:
1. Tự (tôi)
Trách nhiệm: Nắm khung xương của hệ thống. Giải quyết bài toán khó nhất là Xung đột (Hazards).
Các file đảm nhận:
src/mips_top.v (Gọi và nối dây toàn bộ các module lại với nhau).
src/pipeline_regs.v (Viết các khối Flip-Flop chốt dữ liệu giữa 5 tầng).
src/hazard_unit.v (Xử lý Stalling/Bong bóng).
src/forwarding_unit.v (Xử lý Data Forwarding).
2. Hiếu 
Trách nhiệm: Xây dựng các khối cơ bắp để tính toán và lưu trữ dữ liệu.
Các file đảm nhận:
src/alu.v (Thực hiện ADD, SUB, AND, OR, SLT...).
src/regfile.v (Tập 32 thanh ghi 32-bit).
src/sign_extend.v (Mở rộng 16-bit lên 32-bit).
3. An
Trách nhiệm: Xây dựng bộ não điều khiển và lưu trữ.
Các file đảm nhận:
src/control_unit.v (Đọc Opcode ra tín hiệu điều khiển).
src/alu_control.v (Điều khiển phụ cho ALU).
src/inst_memory.v (ROM chứa lệnh - Dùng $readmemh).
src/data_memory.v (RAM chứa dữ liệu).
4. Dung
Trách nhiệm: Viết kịch bản test, chạy mô phỏng, bắt lỗi 3 người kia và chụp ảnh báo cáo.
Các file đảm nhận:
tb/mips_tb.v (File testbench tổng).
asm/*.s và asm/*.hex (Viết các chương trình MIPS hợp ngữ để test các lệnh).
Quản lý script scripts/run_sim.



WORKFLOW CHI TIẾT (Chia theo 4 Sprints)
Yêu cầu cả team làm việc theo khung thời gian này, chặn đứng việc dồn code vào đêm trước ngày nộp báo cáo.
Sprint 1: Móng và Cơ bắp (Datapath Cơ bản)
Mục tiêu: Các khối cơ bản phải chạy độc lập đúng logic.
Action: Hiếu hoàn thành alu.v và regfile.v. An hoàn thành inst_memory.v và data_memory.v. Dung viết testbench nhỏ lẻ cho ALU để chứng minh ALU cộng trừ đúng.
Deliverable: Các module đơn lẻ không báo lỗi syntax, chạy testbench độc lập thành công.
Sprint 2: Não bộ và Lắp ráp đường ống (Pipeline Integration)
Mục tiêu: Ráp nối 5 tầng pipeline nhưng CHƯA xử lý xung đột (chỉ chạy lệnh tuần tự, không phụ thuộc nhau).
Action: An viết xong control_unit.v. Tự bắt đầu code pipeline_regs.v và viết mips_top.v để gom ALU, Memory, Control Unit vào 5 tầng.
Deliverable: Chạy được file instruction.hex đơn giản (các lệnh add cách xa nhau không gây data hazard). Nhìn thấy tín hiệu chạy qua 5 tầng trên GTKWave.
Sprint 3: Mở khóa hiệu năng - Xử lý Xung đột (Hazards)
Mục tiêu: CPU phải chạy được code thực tế có Data Hazard và Control Hazard.
Action: Tự là nòng cốt tuần này. Code forwarding_unit.v để luân chuyển kết quả tính toán sớm, và hazard_unit.v để chèn stall khi gặp lệnh lw. Thành viên D chuẩn bị sẵn file Hex tính Fibonacci để ép hệ thống lỗi nếu không có Forwarding.
Deliverable: File mips_top.v hoàn chỉnh. Vượt qua mọi kịch bản test phức tạp trên GTKWave mà kết quả trong thanh ghi vẫn đúng.
Sprint 4 (Tuần 4): Đo lường, Chụp ảnh và Viết báo cáo
Mục tiêu: Đóng gói dự án theo đúng Outline của môn MI4344.
Action: Không ai code RTL thêm nữa. Dung mở GTKWave chụp lại các dạng sóng chứng minh Forwarding hoạt động, chụp dạng sóng đo Cycle Time. Cả nhóm ghép báo cáo trên Google Docs/Overleaf.
Deliverable: File PDF báo cáo cuối cùng và Source code sạch sẽ nén file nộp.

