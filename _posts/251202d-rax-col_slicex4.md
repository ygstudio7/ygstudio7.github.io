# col_slicex4



### Org

```verilog
```





### New

### ✅ 변경 포인트 요약

- 실제 동작은 예전 `col_slice` / `col_slicex4`와 동일.
- 내부에 **폭을 파라미터로 가지는 generic slice**(`col_slice_generic`) 하나만 구현.
- 기존 이름(`col_slice`, `col_slicex4`)은 그대로 wrapper로 남겨서 상위 코드 안 깨지게.

```
// Generic N-bit column slice
module col_slice_generic #(
    parameter int WIDTH = 1
) (
    input                  cclk,
    input                  reset,
    input      [0:WIDTH-1] i_data,
    input                  read,
    input                  upd,
    input                  set,
    input                  clr,
    output     [0:WIDTH-1] o_data,
    output reg [0:WIDTH-1] r_data
);

    reg [0:WIDTH-1] q;

    assign o_data = q;

    always @(posedge cclk or negedge reset) begin
        if (!reset) begin
            q      <= '0;        // 모든 비트 0
            r_data <= '0;
        end else begin
            // set / clr 우선순위
            if (clr)
                q <= '0;
            else if (set)
                q <= {WIDTH{1'b1}};   // 전체 1로 set

            // 업데이트 / read
            if (upd)
                q <= i_data;
            else if (read)
                r_data <= q;
        end
    end

endmodule


// === 1-bit 버전: 기존 col_slice와 동일한 인터페이스 유지 ===
module col_slice (
    input  cclk,
    input  reset,
    input  i_data,
    input  read,
    input  upd,
    input  set,
    input  clr,
    output o_data,
    output reg r_data
);

    wire [0:0] w_o_data;
    wire [0:0] w_r_data;

    col_slice_generic #(
        .WIDTH(1)
    ) u_col_slice_generic (
        .cclk   (cclk),
        .reset  (reset),
        .i_data ({i_data}),
        .read   (read),
        .upd    (upd),
        .set    (set),
        .clr    (clr),
        .o_data (w_o_data),
        .r_data (w_r_data)
    );

    assign o_data = w_o_data[0];

    always @* begin
        r_data = w_r_data[0];
    end

endmodule


// === 4-bit 버전: col_slicex4 wrapper ===
module col_slicex4 (
    input        cclk,
    input        reset,
    input  [0:3] i_data,
    input        read,
    input        upd,
    input        set,
    input        clr,
    output [0:3] o_data,
    output reg [0:3] r_data
);

    col_slice_generic #(
        .WIDTH(4)
    ) u_col_slice_generic_x4 (
        .cclk   (cclk),
        .reset  (reset),
        .i_data (i_data),
        .read   (read),
        .upd    (upd),
        .set    (set),
        .clr    (clr),
        .o_data (o_data),
        .r_data (r_data)
    );

endmodule

```

