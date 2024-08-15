# SPI-Protocol
\\For Clock Generator Module
module spi_clgen (input wb_clk_in, wb_rst, go, tip, last_clk,
input [`SPI_DIVIDER_LEN-1:0]divider,
output reg sclk_out, cpol_0, cpol_1);

reg [`SPI_DIVIDER_LEN-1:0]cnt;
//Counter counts half period
always @(posedge wb_clk_in or posedge wb_rst)
begin
if(wb_rst)
cnt<={{`SPI_DIVIDER_LEN{1'b0}},1'b1};
else if(tip)
begin
if(cnt==(divider+1))
cnt<={{`SPI_DIVIDER_LEN{1'b0}},1'b1};
else
cnt<=cnt+1;
end
else if(cnt==0)
cnt<={{`SPI_DIVIDER_LEN{1'b0}},1'b1};
end
//Generation of Serial clock
always @(posedge wb_clk_in or posedge wb_rst)
begin
if(wb_rst)
begin
sclk_out<=1'b0;
end
else if(tip)
begin
if(cnt==(divider+1))
begin
if(!last_clk||sclk_out)
sclk_out<=~sclk_out;
end
end
end
//Posedge and Negedge detection of sclk
always @(posedge wb_clk_in or posedge wb_rst)
begin

if(wb_rst)
begin
cpol_0<=1'b0;
cpol_1<=1'b0;
end
else
begin
cpol_0<=1'b0;
cpol_1<=1'b0;
if(tip)
begin
if(~sclk_out)
begin
if(cnt==divider)
begin
cpol_0<=1;
end
end
end
if(tip)
begin
if(sclk_out)
begin
if(cnt==divider)
begin
cpol_1<=1;
end
end
end
end
end
endmodule

\\For Shift Registers
begin
master_data[15:8] <= p_in [15:8];
end
if(byte_sel[2])
begin
master_data[23:16] <= p_in [23:16];
end
if(byte_sel[3])
begin
master_data[31:24] <= p_in [31:24];
end
end
else if(latch[1] && !tip) //TX1 is selected
begin
if(byte_sel[0])
begin
master_data[39:32] <= p_in [7:0];
end
if(byte_sel[1])
begin
master_data[47:40] <= p_in [15:8];
end
if(byte_sel[2])
begin
master_data[55:48] <= p_in [23:16];
end
if(byte_sel[3])
begin
master_data[63:56] <= p_in [31:24];
end
end
else if(latch[2] && !tip) //TX2 is selected..
begin
if(byte_sel[0])
begin
master_data[71:64] <= p_in [7:0];
end
if(byte_sel[1])
begin
master_data[79:72] <= p_in [15:8];
end
if(byte_sel[2])
begin
master_data[87:80] <= p_in [23:16];
end
if(byte_sel[3])
begin
master_data[95:88] <= p_in [31:24];
end
end
else if(latch[3] && !tip) //TX3 is selected..
begin
if(byte_sel[0])
begin
master_data[103:96] <= p_in[7:0];
end
if(byte_sel[1])
begin
master_data[111:104] <= p_in[15:8];
end
if(byte_sel[2])
begin
master_data[119:112] <= p_in[23:16];
end
if(byte_sel[3])
begin
master_data[127:120] <= p_in[31:24];
end
end
`else
`ifdef SPI_MAX_CHAR_64
else if(latch[0] && !tip) //TX0 is selected..
begin
if(byte_sel[0])
begin
master_data[7:0] <= p_in[7:0];
end
if(byte_sel[1])
begin
master_data[15:8] <= p_in[15:8];
end
if(byte_sel[2])
begin
master_data[23:16] <= p_in[23:16];
end
if(byte_sel[3])
begin
master_data[31:24] <= p_in[31:24];
end
end
else if(latch[1] && !tip)//TX1 is selected..
begin
if(byte_sel[0])
begin
master_data[39:32] <= p_in[7:0];
end
if(byte_sel[1])
begin
master_data[47:40] <= p_in[15:8];
end
if(byte_sel[2])
begin
master_data[55:48] <= p_in[23:16];
end
if(byte_sel[3])
begin
master_data[63:56] <= p_in[31:24];
end
end
`else
else if(latch[0] && !tip)//TX0 is selected..
begin
`ifdef SPI_MAX_CHAR_8
if(byte_sel[0])
begin
master_data[7:0] <= p_in[7:0];
end
`endif
`ifdef SPI_MAX_CHAR_16
if(byte_sel[0])
begin
master_data[7:0] <= p_in[7:0];
end
if(byte_sel[1])
begin
master_data[15:8] <= p_in[7:0];
end
`endif
`ifdef SPI_MAX_CHAR_24
if(byte_sel[0])
begin
master_data[7:0] <= p_in[7:0];
end
if(byte_sel[1])
begin
master_data[15:8] <= p_in[7:0];
end
if(byte_sel[2])
begin
master_data[23:16] <= p_in[23:16];
end
`endif
`ifdef SPI_MAX_CHAR_32
if(byte_sel[0])
begin
master_data[7:0] <= p_in[7:0];
end
if(byte_sel[1])
begin
master_data[15:8] <= p_in[7:0]; //I think here it should be 15:8
end
if(byte_sel[2])
begin
master_data[23:16] <= p_in[23:16];
end
if(byte_sel[3])
begin
master_data[31:24] <= p_in[31:24];
end
`endif
end
`endif
`endif
//Receiving bit from serial line................
else
begin
if(rx_clk)
begin master_data[rx_bit_pos[`SPI_CHAR_LEN_BITS-1:0]] <= miso;
end
end
end
endmodule

\\Verilog code for spi Wishbone Master:
module wishbone_master(input clk_in, rst_in, ack_in, err_in, input [31:0]dat_in,
output reg [4:0]adr_o, output reg cyc_o, stb_o, we_o,
output reg [31:0]dat_o, output reg [3:0]sel_o) ;
//Internal Signals...............................................
integer adr_temp, sel_temp, dat_temp;
reg we_temp, cyc_temp, stb_temp;
//Initialise task.............................................
task initialize;
begin
{adr_temp, cyc_temp, stb_temp, we_temp, dat_temp, sel_temp}=0;
end
endtask
//Wishbone Bus Cycles Single Read/Write
task single_write;
input [4:0]adr;
input [31:0]dat;
input [3:0]sel;
begin
@(negedge clk_in);
adr_temp = adr;
sel_temp = sel;
we_temp = 1;
dat_temp = dat;
cyc_temp = 1;
stb_temp = 1;
@(negedge clk_in);
wait(~ack_in)
@(negedge clk_in);
adr_temp = 5'dz;
sel_temp = 4'd0;
we_temp = 1'b0;
dat_temp = 32'dz;
cyc_temp = 1'b0;
stb_temp = 1'b0;
end
endtask
always @(posedge clk_in)
begin
adr_o <= adr_temp;
end
always @(posedge clk_in)
begin
we_o <= we_temp;
end
always @(posedge clk_in)
begin
dat_o <= dat_temp;
end
always @(posedge clk_in)
begin
sel_o <= sel_temp;
end
always @(posedge clk_in)
begin
if(rst_in)
cyc_o <= 0;
else
cyc_o <= cyc_temp;
end
always @(posedge clk_in)
begin
if(rst_in)
stb_o <= 0;
else
stb_o <= stb_temp;
end
endmodule

\\Verilog Code for Slave:
module spi_slave(sclk,mosi,ss_pad_o,miso);
input sclk,mosi;
input [`SPI_SS_NB-1:0]ss_pad_o;
output miso;
reg rx_slave =1'b0;
reg tx_slave=1'b0;
reg[127:0]temp1=0;
reg[127:0]temp2=0;
reg miso1=1'b0;
reg miso2=1'b1;
always@(posedge sclk)
begin
if((ss_pad_o != 8'b11111111) && ~rx_slave && tx_slave)
begin
temp1<={temp1[126:0],mosi};
end
end
always@(negedge sclk)
begin
if((ss_pad_o != 8'b11111111) && rx_slave && ~tx_slave)
begin
temp2<={temp2[126:0],mosi};
end
end
always@(negedge sclk)
begin
if(rx_slave && ~tx_slave)
begin
miso1<=temp1[127];
end
end
always@(posedge sclk)
begin
if(~rx_slave && tx_slave)
begin
miso2<=temp2[127];
end
end
assign miso= miso1 || miso2;
endmodule

\\SPI TEST BENCH:
module tb;
reg wb_clk_in, wb_rst_in;
wire wb_we_in,wb_stb_in, wb_cyc_in,miso;
wire [4:0] wb_adr_in;
wire [31:0] wb_dat_in;
wire [3:0] wb_sel_in;
wire [31:0] wb_dat_o;
wire wb_ack_out, wb_int_o, sclk_out, mosi;
wire [`SPI_SS_NB-1:0] ss_pad_o;
parameter T=20;
wishbone_master MASTER(wb_clk_in,wb_rst_in, wb_ack_out, wb_err_in, wb_dat_o,
wb_adr_in, wb_cyc_in, wb_stb_in, wb_we_in, wb_dat_in, wb_sel_in);
spi_top SPI_CORE(wb_clk_in,wb_rst_in, wb_adr_in, wb_dat_o, wb_sel_in,
wb_we_in,wb_stb_in,wb_cyc_in,wb_ack_out,wb_int_o,
wb_dat_in,ss_pad_o,sclk_out,mosi,miso);
spi_slave SLAVE(sclk_out,mosi,ss_pad_o,miso);
initial
begin
wb_clk_in =1'b0;
forever
#(T/2) wb_clk_in = ~wb_clk_in;
end
task rst;
begin
wb_rst_in =1'b1;
#13;
wb_rst_in=1'b0;
end
endtask
//tx_neg=1, rx_neg=0,lsb=1,char_len=4
37
/*initial
begin
rst;
//initialize the WISHBONE output signals
MASTER initialize;
//configure control register with go_busy being low
MASTER.single_write(5'h10,32'h0000_3C04,4'b1111);
//configure divider with go_busy being low
MASTER.single_write(5'h14,32'h0000_0004,4'b1111);
//configure slave register with go_busy being low
MASTER.single_write(5'h18,32'h0000_0001,4'b1111);
//configure tx register with go busy being low and processor is sending 4 bits
MASTER.single_write(5'h00,32'h0000_236f,4'b1111);
//configure control register with go busy being high
MASTER.single_write(5'h10,32'h0000_3d04,4'b1111);
repeat(100)
@(negedge wb_clk_in);
$finish;
#10000
$finish;
end*/
//tx_neg=1, rx_neg=0, LSB =0, char_len=4
/*initial
begin
rst;
//initialize the WISHBONE output signals
MASTER.initialize;
//configure control register with go busy being low
MASTER.single_write(5'h10, 32'h0000_3404, 4'b1111);
//configure slave register with go busy being low
MASTER.single_write(5'h14, 32'h0000_0004, 4'b1111);
//configure slave register with go busy being low
MASTER.single_write(5'h18, 32'h0000_0001, 4'b1111);
//configure the tx register with go busy being low and processor is sending 4 bits
MASTER.single_write(5'h00, 32'h0000_26ff, 4'b1111);
//configure control register with go_busy high
MASTER.single_write(5'h10, 32'h0000_3504, 4'b1111);
repeat(100)
@(negedge wb_clk_in);
$finish;
#10000
$finish;
end*/
//tx_neg=0, rx_neg=1, LSB=1,char_len=4
/*initial
begin
rst;
//initialize the WISHBONE output signals
MASTER.initialize;
//configure control register with go_busy low
MASTER.single_write(5'h10,32'h0000_3A04,4'b1111);
//configure divider with go_busy low
MASTER.single_write(5'h14,32'h0000_0004,4'b1111);
//configure slave register with go_busy being low
MASTER.single_write(5'h18,32'h0000_0001,4'b1111);
//configure tx register with go_busy being low and processor is sending 4 bits
MASTER.single_write(5'h00,32'h0000_236f,4'b1111);
//configure control register with go_busy being high
MASTER.single_write(5'h10,32'h0000_3B04,4'b1111);
repeat(100)
@(negedge wb_clk_in);
//$finish;
#10000
//$finish;*/
//tx_neg =0, rx_neg = 1,LSB=0, char_len=4
initial
begin
rst;
//initialize the WISHBONE output signals
MASTER.initialize;
//configure control register with go_busy low
MASTER.single_write(5'h10,32'h0000_3204,4'b1111);
//configure divider with go_busy low
MASTER.single_write(5'h14,32'h0000_0004,4'b1111);
//configure slave register with go_busy being low
MASTER.single_write(5'h18,32'h0000_0001,4'b1111);
//configure tx register with go_busy being low and processor is sending 4 bits
MASTER.single_write(5'h00,32'h0000_236f,4'b1111);
//configure control register with go_busy being high
MASTER.single_write(5'h10,32'h0000_3304,4'b1111);
repeat(100)
@(negedge wb_clk_in);
$finish;
#10000
$finish;
end
initial
begin
/*//tx_neg=1, rx_neg = 0
SLAVE.rx_slave=0;
SLAVE.tx_slave=1;*/
//tx_neg=0, rx_neg=1
SLAVE.rx_slave=1;
SLAVE.tx_slave=0;
end
endmodule
