```
```

## 3. 트랜잭션 정의 (pixel 단위)

`_im_sv_intf.p` 안에 이미 struct가 있으니까, scoreboard 입장에선 “한 번의 `_ACT`=1일 때를 한 sample/pixel 트랜잭션”으로 보는 게 자연스러움.

```
// im_rhdr_item.sv
class im_rhdr_item extends uvm_sequence_item;
  rand int unsigned  vcnt;
  rand int unsigned  hcnt;
  rand bit           vs;
  rand bit           hs;
  rand bit           act;
  rand bit [31:0]    dat;      // DW 32 or 그냥 int unsigned로

  string             fname;    // frame 이름 (p._fname)

  // 필요하면 L, A_R/G/B, J_R/G/B 등 internal stream도 위한 필드 추가 가능

  `uvm_object_utils_begin(im_rhdr_item)
    `uvm_field_int(vcnt, UVM_DEFAULT)
    `uvm_field_int(hcnt, UVM_DEFAULT)
    `uvm_field_int(vs,   UVM_DEFAULT)
    `uvm_field_int(hs,   UVM_DEFAULT)
    `uvm_field_int(act,  UVM_DEFAULT)
    `uvm_field_int(dat,  UVM_DEFAULT)
    `uvm_field_string(fname, UVM_DEFAULT)
  `uvm_object_utils_end

  function new(string name="im_rhdr_item");
    super.new(name);
  endfunction
endclass
```

DW 가 48/32/real 섞여 있긴 하지만, **scoreboard에서 golden file과 비교할 건 최종 SVO(DW=48)** 한 가지만 먼저 잡는 게 좋음. (필요하면 real용 전용 item 만들 수도 있고.)