# UVM入门实验0
- 在验证顶层需要将uvm package导入
```verilog
import uvm_pkg::*;
`include "uvm_macros.svh"
```
- 9个phase都执行完后UVM会立即退出，不会继续执行剩下的内容（run_test()后面的内容），所以要合理应用 `phase.raise_objection` 和 `phase.drop_objection`

```verilog
package test_pkg;
  import uvm_pkg::*;
  `include "uvm_macros.svh"

  class top extends uvm_test;
    `uvm_component_utils(top)
    function new(string name = "top", uvm_component parent = null);
      super.new(name, parent);
      `uvm_info("UVM_TOP", "SV TOP creating", UVM_LOW)
    endfunction
    task run_phase(uvm_phase phase);
      phase.raise_objection(this);
      `uvm_info("UVM_TOP", "test is running", UVM_LOW)
      phase.drop_objection(this);
    endtask
  endclass

endpackage

module uvm_test_inst;

  import uvm_pkg::*;
  `include "uvm_macros.svh"
  import test_pkg::*;


  initial begin
    `uvm_info("UVM_TOP", "test started", UVM_LOW)
    run_test("top");// uvm_top.run_test("top");
    `uvm_info("UVM_TOP", "test finished", UVM_LOW)
  end

endmodule
```

# UVM入门实验1
## 组件创建
```verilog
package factory_pkg;
  import uvm_pkg::*;
  `include "uvm_macros.svh"

  class trans extends uvm_object;
    bit[31:0] data;
    `uvm_object_utils(trans)
    function new(string name = "trans");
      super.new(name);
      `uvm_info("CREATE", $sformatf("trans type [%s] created", name), UVM_LOW)
    endfunction
  endclass

  class bad_trans extends trans;
    bit is_bad = 1;
    `uvm_object_utils(bad_trans)
    function new(string name = "trans");
      super.new(name);
      `uvm_info("CREATE", $sformatf("bad_trans type [%s] created", name), UVM_LOW)
    endfunction
  endclass

  class unit extends uvm_component;
    `uvm_component_utils(unit)
    function new(string name = "unit", uvm_component parent = null);
      super.new(name, parent);
      `uvm_info("CREATE", $sformatf("unit type [%s] created", name), UVM_LOW)
    endfunction
  endclass

  class big_unit extends unit;
    bit is_big = 1;
    `uvm_component_utils(big_unit)
    function new(string name = "bit_unit", uvm_component parent = null);
      super.new(name, parent);
      `uvm_info("CREATE", $sformatf("big_unit type [%s] created", name), UVM_LOW)
    endfunction
  endclass

  class top extends uvm_test;
    `uvm_component_utils(top)
    function new(string name = "top", uvm_component parent = null);
      super.new(name, parent);
    endfunction
    function void build_phase(uvm_phase phase);
      super.build_phase(phase);
    endfunction
    task run_phase(uvm_phase phase);
      phase.raise_objection(this);
      #1us;
      phase.drop_objection(this);
    endtask
  endclass

  class object_create extends top;
    trans t1, t2, t3, t4;
    `uvm_component_utils(object_create)
    function new(string name = "object_create", uvm_component parent = null);
      super.new(name, parent);
    endfunction    
    function void build_phase(uvm_phase phase);
      uvm_factory f = uvm_factory::get(); 
      super.build_phase(phase);
      t1 = new("t1");
      t2 = trans::type_id::create("t2", this);
      // this表示t2的上一层，而不是父类
      // type_id: typedef uvm_object_registry #(cmd) type_id;
      // create 是 uvm_component_registry #(T,Tname) 的静态函数
      void'($cast(t3,f.create_object_by_type(trans::get_type(), get_full_name(), "t3")));
      // pure virtual function uvm_object create_object_by_type (
   	  //        uvm_object_wrapper 	requested_type,	  	
   	  //        string 	parent_inst_path	 = 	"",
   	  //        string 	name	 = 	""
      //        )
      //
      // get_type()是一个静态函数，static function uvm_object_wrapper get_type ()
      // 这里的get_full_name 是 uvm_componen 中的函数，返回当前路径字符串
      void'($cast(t4,create_object("trans", "t4"))); // pre-defined method inside component
      // function uvm_object create_object (
      //   	string 	requested_type_name,	  	
   	  //    string 	name	 = 	""
      //    )
      // 这里create_object是uvm_component中的函数
    endfunction
  endclass

  class object_override extends object_create;
    `uvm_component_utils(object_override)
    function new(string name = "object_override", uvm_component parent = null);
      super.new(name, parent);
    endfunction
    function void build_phase(uvm_phase phase);
      set_type_override_by_type(trans::get_type(), bad_trans::get_type());
      // 这个函数是按类型覆盖
      super.build_phase(phase);
    endfunction
  endclass

  class component_create extends top;
    unit u1, u2, u3, u4;
    `uvm_component_utils(component_create)
    function new(string name = "component_create", uvm_component parent = null);
      super.new(name, parent);
    endfunction
    function void build_phase(uvm_phase phase);
      uvm_factory f = uvm_factory::get(); 
      super.build_phase(phase);
      u1 = new("u1"); 
      u2 = unit::type_id::create("u2", this)；
      void'($cast(u3,f.create_component_by_type(unit::get_type(), get_full_name(), "u3", this)));
      // pure virtual function uvm_component create_component_by_type (
   	  //    uvm_object_wrapper 	requested_type,	  	
   	  //    string 	parent_inst_path	 = 	"",
   	  //    string 	name,	  	
   	  //    uvm_component 	parent	  	
      //    )
      void'($cast(u4,create_component("unit", "u4"))); 
    endfunction
  endclass

  class component_override extends component_create;
    `uvm_component_utils(component_override)
    function new(string name = "component_override", uvm_component parent = null);
      super.new(name, parent);
    endfunction
    function void build_phase(uvm_phase phase);
      set_type_override("unit", "big_unit");
      // 这个函数是按名字覆盖
      super.build_phase(phase);
    endfunction
  endclass
  
endpackage

module factory_mechanism;

  import uvm_pkg::*;
  `include "uvm_macros.svh"
  import factory_pkg::*;

  initial begin
    run_test(""); 
  end

endmodule
```

# UVM入门实验1
## object的方法
```verilog
package object_methods_pkg;
  import uvm_pkg::*;
  `include "uvm_macros.svh"
  
  typedef enum {WRITE, READ, IDLE} op_t;  

  class trans extends uvm_object;
    bit[31:0] addr;
    bit[31:0] data;
    op_t op;
    string name;
    `uvm_object_utils_begin(trans)
      `uvm_field_int(addr, UVM_ALL_ON)
      `uvm_field_int(data, UVM_ALL_ON)
      `uvm_field_enum(op_t, op, UVM_ALL_ON)
      `uvm_field_string(name, UVM_ALL_ON)
    `uvm_object_utils_end
    function new(string name = "trans");
      super.new(name);
      `uvm_info("CREATE", $sformatf("trans type [%s] created", name), UVM_LOW)
    endfunction
    function bit do_compare(uvm_object rhs, uvm_comparer comparer); // compare的回调函数
      trans t;
      do_compare = 1;
      void'($cast(t, rhs));
      if(addr != t.addr) begin
        do_compare = 0;
        `uvm_warning("CMPERR", $sformatf("addr %8x != %8x", addr, t.addr))
      end
      if(data != t.data) begin
        do_compare = 0;
        `uvm_warning("CMPERR", $sformatf("data %8x != %8x", data, t.data))
      end
      if(op != t.op) begin
        do_compare = 0;
        `uvm_warning("CMPERR", $sformatf("op %s != %8x", op, t.op))
      end
      if(addr != t.addr) begin
        do_compare = 0;
        `uvm_warning("CMPERR", $sformatf("name %8x != %8x", name, t.name))
      end
    endfunction
  endclass


  class object_methods_test extends uvm_test;
    `uvm_component_utils(object_methods_test)
    function new(string name = "object_methods_test", uvm_component parent = null);
      super.new(name, parent);
    endfunction
    function void build_phase(uvm_phase phase);
      super.build_phase(phase);
    endfunction
    task run_phase(uvm_phase phase);
      trans t1, t2;
      bit is_equal;
      phase.raise_objection(this);
      t1 = trans::type_id::create("t1");
      t1.data = 'h1FF;
      t1.addr = 'hF100;
      t1.op = WRITE;
      t1.name = "t1";
      t2 = trans::type_id::create("t2");
      t2.data = 'h2FF;
      t2.addr = 'hF200;
      t2.op = WRITE;
      t2.name = "t2";
      is_equal = t1.compare(t2);
      uvm_default_comparer.show_max = 10;
      is_equal = t1.compare(t2);
      /*
        UVM_INFO @ 0: reporter [RNTST] Running test object_methods_test...
        UVM_INFO uvm_object_methods_ref.sv(21) @ 0: reporter [CREATE] trans type [t1] created
        UVM_INFO uvm_object_methods_ref.sv(21) @ 0: reporter [CREATE] trans type [t2] created
        UVM_INFO /opt/synopsys/vcs/vcs-mx/O-2018.09-SP2/etc/uvm-1.2/base/uvm_comparer.svh(351) @ 0: reporter [MISCMP] Miscompare for t1.addr: lhs = 'hf100 : rhs = 'hf200
        UVM_WARNING uvm_object_methods_ref.sv(29) @ 0: reporter [CMPERR] addr 0000f100 != 0000f200
        UVM_WARNING uvm_object_methods_ref.sv(33) @ 0: reporter [CMPERR] data 000001ff != 000002ff
        UVM_WARNING uvm_object_methods_ref.sv(41) @ 0: reporter [CMPERR] name 00007431 != 00007432
        UVM_INFO /opt/synopsys/vcs/vcs-mx/O-2018.09-SP2/etc/uvm-1.2/base/uvm_comparer.svh(382) @ 0: reporter [MISCMP] 1 Miscompare(s) for object t2@360 vs. t1@357
        UVM_INFO /opt/synopsys/vcs/vcs-mx/O-2018.09-SP2/etc/uvm-1.2/base/uvm_comparer.svh(351) @ 0: reporter [MISCMP] Miscompare for t1.addr: lhs = 'hf100 : rhs = 'hf200
        UVM_INFO /opt/synopsys/vcs/vcs-mx/O-2018.09-SP2/etc/uvm-1.2/base/uvm_comparer.svh(351) @ 0: reporter [MISCMP] Miscompare for t1.data: lhs = 'h1ff : rhs = 'h2ff
        UVM_INFO /opt/synopsys/vcs/vcs-mx/O-2018.09-SP2/etc/uvm-1.2/base/uvm_comparer.svh(351) @ 0: reporter [MISCMP] Miscompare for t1.name: lhs = "t1" : rhs = "t2"
        UVM_WARNING uvm_object_methods_ref.sv(29) @ 0: reporter [CMPERR] addr 0000f100 != 0000f200
        UVM_WARNING uvm_object_methods_ref.sv(33) @ 0: reporter [CMPERR] data 000001ff != 000002ff
        UVM_WARNING uvm_object_methods_ref.sv(41) @ 0: reporter [CMPERR] name 00007431 != 00007432
        UVM_INFO /opt/synopsys/vcs/vcs-mx/O-2018.09-SP2/etc/uvm-1.2/base/uvm_comparer.svh(382) @ 0: reporter [MISCMP] 3 Miscompare(s) for object t2@360 vs. t1@357
        UVM_WARNING uvm_object_methods_ref.sv(74) @ 0: uvm_test_top [CMPERR] t1 is not equal to t2
        UVM_INFO uvm_object_methods_ref.sv(78) @ 0: uvm_test_top [COPY] Before uvm_object copy() taken
      */
      if(!is_equal)
        `uvm_warning("CMPERR", "t1 is not equal to t2")
      else
        `uvm_info("CMPERR", "t1 is equal to t2", UVM_LOW)
        
      `uvm_info("COPY", "Before uvm_object copy() taken", UVM_LOW)
      t1.print();
      t2.print();
      `uvm_info("COPY", "After uvm_object t2 is copied to t1", UVM_LOW)
      t1.copy(t2);
      t1.print();
      t2.print();
      /*------------------------------
        Name    Type      Size  Value 
        ------------------------------
        t1      trans     -     @357  
          addr  integral  32    'hf100
          data  integral  32    'h1ff 
          op    op_t      32    WRITE 
          name  string    2     t1    
        ------------------------------
        ------------------------------
        Name    Type      Size  Value 
        ------------------------------
        t2      trans     -     @360  
          addr  integral  32    'hf200
          data  integral  32    'h2ff 
          op    op_t      32    WRITE 
          name  string    2     t2    
        ------------------------------
        UVM_INFO uvm_object_methods_ref.sv(81) @ 0: uvm_test_top [COPY] After uvm_object t2 is copied to t1
        ------------------------------
        Name    Type      Size  Value 
        ------------------------------
        t1      trans     -     @357  
          addr  integral  32    'hf200
          data  integral  32    'h2ff 
          op    op_t      32    WRITE 
          name  string    2     t2    
        ------------------------------
        ------------------------------
        Name    Type      Size  Value 
        ------------------------------
        t2      trans     -     @360  
          addr  integral  32    'hf200
          data  integral  32    'h2ff 
          op    op_t      32    WRITE 
          name  string    2     t2    
        ------------------------------ */
      `uvm_info("CMP", "Compare t1 and t2", UVM_LOW)
      is_equal = t1.compare(t2);
      if(!is_equal)
        `uvm_warning("CMPERR", "t1 is not equal to t2")
      else
        `uvm_info("CMPERR", "t1 is equal to t2", UVM_LOW)       
      
      #1us;
      phase.drop_objection(this);
    endtask
  endclass
  
endpackage

module uvm_object_methods_ref;

  import uvm_pkg::*;
  `include "uvm_macros.svh"
  import object_methods_pkg::*;

  initial begin
    run_test(""); // empty test name
  end

endmodule
```

## phase的顺序
```verilog
package phase_order_pkg;
  import uvm_pkg::*;
  `include "uvm_macros.svh"

  class comp2 extends uvm_component;
    `uvm_component_utils(comp2)
    function new(string name = "comp2", uvm_component parent = null);
      super.new(name, parent);
      `uvm_info("CREATE", $sformatf("unit type [%s] created", name), UVM_LOW)
    endfunction
    function void build_phase(uvm_phase phase);
      super.build_phase(phase);
      `uvm_info("BUILD", "comp2 build phase entered", UVM_LOW)
      `uvm_info("BUILD", "comp2 build phase exited", UVM_LOW)
    endfunction
    function void connect_phase(uvm_phase phase);
      super.connect_phase(phase);
      `uvm_info("CONNECT", "comp2 connect phase entered", UVM_LOW)
      `uvm_info("CONNECT", "comp2 connect phase exited", UVM_LOW)
    endfunction
    task run_phase(uvm_phase phase);
      super.run_phase(phase);
      `uvm_info("RUN", "comp2 run phase entered", UVM_LOW)
      `uvm_info("RUN", "comp2 run phase exited", UVM_LOW)
    endtask
    function void report_phase(uvm_phase phase);
      super.report_phase(phase);
      `uvm_info("REPORT", "comp2 report phase entered", UVM_LOW)
      `uvm_info("REPORT", "comp2 report phase exited", UVM_LOW)   
    endfunction
  endclass
  
  class comp3 extends uvm_component;
    `uvm_component_utils(comp3)
    function new(string name = "comp3", uvm_component parent = null);
      super.new(name, parent);
      `uvm_info("CREATE", $sformatf("unit type [%s] created", name), UVM_LOW)
    endfunction
    function void build_phase(uvm_phase phase);
      super.build_phase(phase);
      `uvm_info("BUILD", "comp3 build phase entered", UVM_LOW)
      `uvm_info("BUILD", "comp3 build phase exited", UVM_LOW)
    endfunction
    function void connect_phase(uvm_phase phase);
      super.connect_phase(phase);
      `uvm_info("CONNECT", "comp3 connect phase entered", UVM_LOW)
      `uvm_info("CONNECT", "comp3 connect phase exited", UVM_LOW)
    endfunction
    task run_phase(uvm_phase phase);
      super.run_phase(phase);
      `uvm_info("RUN", "comp3 run phase entered", UVM_LOW)
      `uvm_info("RUN", "comp3 run phase exited", UVM_LOW)
    endtask
    function void report_phase(uvm_phase phase);
      super.report_phase(phase);
      `uvm_info("REPORT", "comp3 report phase entered", UVM_LOW)
      `uvm_info("REPORT", "comp3 report phase exited", UVM_LOW)   
    endfunction
  endclass
  
  class comp1 extends uvm_component;
    comp2 c2;
    comp3 c3;
    `uvm_component_utils(comp1)
    function new(string name = "comp1", uvm_component parent = null);
      super.new(name, parent);
      `uvm_info("CREATE", $sformatf("unit type [%s] created", name), UVM_LOW)
    endfunction
    function void build_phase(uvm_phase phase);
      super.build_phase(phase);
      `uvm_info("BUILD", "comp1 build phase entered", UVM_LOW)
      c2 = comp2::type_id::create("c2", this);
      c3 = comp3::type_id::create("c3", this);
      `uvm_info("BUILD", "comp1 build phase exited", UVM_LOW)
    endfunction
    function void connect_phase(uvm_phase phase);
      super.connect_phase(phase);
      `uvm_info("CONNECT", "comp1 connect phase entered", UVM_LOW)
      `uvm_info("CONNECT", "comp1 connect phase exited", UVM_LOW)
    endfunction
    task run_phase(uvm_phase phase);
      super.run_phase(phase);
      `uvm_info("RUN", "comp1 run phase entered", UVM_LOW)
      `uvm_info("RUN", "comp1 run phase exited", UVM_LOW)
    endtask
    function void report_phase(uvm_phase phase);
      super.report_phase(phase);
      `uvm_info("REPORT", "comp1 report phase entered", UVM_LOW)
      `uvm_info("REPORT", "comp1 report phase exited", UVM_LOW)   
    endfunction
  endclass

  class phase_order_test extends uvm_test;
    comp1 c1;
    `uvm_component_utils(phase_order_test)
    function new(string name = "phase_order_test", uvm_component parent = null);
      super.new(name, parent);
    endfunction
    function void build_phase(uvm_phase phase);
      super.build_phase(phase);
      `uvm_info("BUILD", "phase_order_test build phase entered", UVM_LOW)
      c1 = comp1::type_id::create("c1", this);
      `uvm_info("BUILD", "phase_order_test build phase exited", UVM_LOW)
    endfunction
    function void connect_phase(uvm_phase phase);
      super.connect_phase(phase);
      `uvm_info("CONNECT", "phase_order_test connect phase entered", UVM_LOW)
      `uvm_info("CONNECT", "phase_order_test connect phase exited", UVM_LOW)
    endfunction
    task run_phase(uvm_phase phase);
      super.run_phase(phase);
      `uvm_info("RUN", "phase_order_test run phase entered", UVM_LOW)
      phase.raise_objection(this);
      #1us;
      phase.drop_objection(this);
      `uvm_info("RUN", "phase_order_test run phase exited", UVM_LOW)
    endtask
    function void report_phase(uvm_phase phase);
      super.report_phase(phase);
      `uvm_info("REPORT", "phase_order_test report phase entered", UVM_LOW)
      `uvm_info("REPORT", "phase_order_test report phase exited", UVM_LOW)    
    endfunction
    
    task reset_phase(uvm_phase phase);
      `uvm_info("RESET", "phase_order_test reset phase entered", UVM_LOW)
      phase.raise_objection(this);
      #1us;
      phase.drop_objection(this);
      `uvm_info("RESET", "phase_order_test reset phase exited", UVM_LOW)
    endtask
    
    task main_phase(uvm_phase phase);
      `uvm_info("MAIN", "phase_order_test main phase entered", UVM_LOW)
      phase.raise_objection(this);
      #1us;
      phase.drop_objection(this);
      `uvm_info("MAIN", "phase_order_test main phase exited", UVM_LOW)
    endtask 
  endclass
endpackage

module phase_order_ref;

  import uvm_pkg::*;
  `include "uvm_macros.svh"
  import phase_order_pkg::*;

  initial begin
    run_test(""); // empty test name
  end

endmodule
/*
UVM_INFO @ 0: reporter [RNTST] Running test phase_order_test...
UVM_INFO phase_order_ref.sv(102) @ 0: uvm_test_top [BUILD] phase_order_test build phase entered
UVM_INFO phase_order_ref.sv(68) @ 0: uvm_test_top.c1 [CREATE] unit type [c1] created
UVM_INFO phase_order_ref.sv(104) @ 0: uvm_test_top [BUILD] phase_order_test build phase exited
UVM_INFO phase_order_ref.sv(72) @ 0: uvm_test_top.c1 [BUILD] comp1 build phase entered
UVM_INFO phase_order_ref.sv(10) @ 0: uvm_test_top.c1.c2 [CREATE] unit type [c2] created
UVM_INFO phase_order_ref.sv(38) @ 0: uvm_test_top.c1.c3 [CREATE] unit type [c3] created
UVM_INFO phase_order_ref.sv(75) @ 0: uvm_test_top.c1 [BUILD] comp1 build phase exited
UVM_INFO phase_order_ref.sv(14) @ 0: uvm_test_top.c1.c2 [BUILD] comp2 build phase entered
UVM_INFO phase_order_ref.sv(15) @ 0: uvm_test_top.c1.c2 [BUILD] comp2 build phase exited
UVM_INFO phase_order_ref.sv(42) @ 0: uvm_test_top.c1.c3 [BUILD] comp3 build phase entered
UVM_INFO phase_order_ref.sv(43) @ 0: uvm_test_top.c1.c3 [BUILD] comp3 build phase exited
UVM_INFO phase_order_ref.sv(19) @ 0: uvm_test_top.c1.c2 [CONNECT] comp2 connect phase entered
UVM_INFO phase_order_ref.sv(20) @ 0: uvm_test_top.c1.c2 [CONNECT] comp2 connect phase exited
UVM_INFO phase_order_ref.sv(47) @ 0: uvm_test_top.c1.c3 [CONNECT] comp3 connect phase entered
UVM_INFO phase_order_ref.sv(48) @ 0: uvm_test_top.c1.c3 [CONNECT] comp3 connect phase exited
UVM_INFO phase_order_ref.sv(79) @ 0: uvm_test_top.c1 [CONNECT] comp1 connect phase entered
UVM_INFO phase_order_ref.sv(80) @ 0: uvm_test_top.c1 [CONNECT] comp1 connect phase exited
UVM_INFO phase_order_ref.sv(108) @ 0: uvm_test_top [CONNECT] phase_order_test connect phase entered
UVM_INFO phase_order_ref.sv(109) @ 0: uvm_test_top [CONNECT] phase_order_test connect phase exited
UVM_INFO phase_order_ref.sv(24) @ 0: uvm_test_top.c1.c2 [RUN] comp2 run phase entered
UVM_INFO phase_order_ref.sv(25) @ 0: uvm_test_top.c1.c2 [RUN] comp2 run phase entered
UVM_INFO phase_order_ref.sv(52) @ 0: uvm_test_top.c1.c3 [RUN] comp3 run phase entered
UVM_INFO phase_order_ref.sv(53) @ 0: uvm_test_top.c1.c3 [RUN] comp3 run phase entered
UVM_INFO phase_order_ref.sv(84) @ 0: uvm_test_top.c1 [RUN] comp1 run phase entered
UVM_INFO phase_order_ref.sv(85) @ 0: uvm_test_top.c1 [RUN] comp1 run phase entered
UVM_INFO phase_order_ref.sv(113) @ 0: uvm_test_top [RUN] phase_order_test run phase entered
UVM_INFO phase_order_ref.sv(126) @ 0: uvm_test_top [RESET] phase_order_test reset phase entered
UVM_INFO phase_order_ref.sv(117) @ 1000000: uvm_test_top [RUN] phase_order_test run phase exited
UVM_INFO phase_order_ref.sv(130) @ 1000000: uvm_test_top [RESET] phase_order_test reset phase exited
UVM_INFO /opt/synopsys/vcs/vcs-mx/O-2018.09-SP2/etc/uvm-1.2/base/uvm_objection.svh(1276) @ 1000000: reporter [TEST_DONE] 'run' phase is ready to proceed to the 'extract' phase
UVM_INFO phase_order_ref.sv(134) @ 1000000: uvm_test_top [MAIN] phase_order_test main phase entered
UVM_INFO phase_order_ref.sv(138) @ 2000000: uvm_test_top [MAIN] phase_order_test main phase exited
UVM_INFO phase_order_ref.sv(29) @ 2000000: uvm_test_top.c1.c2 [REPORT] comp2 report phase entered
UVM_INFO phase_order_ref.sv(30) @ 2000000: uvm_test_top.c1.c2 [REPORT] comp2 report phase exited
UVM_INFO phase_order_ref.sv(57) @ 2000000: uvm_test_top.c1.c3 [REPORT] comp3 report phase entered
UVM_INFO phase_order_ref.sv(58) @ 2000000: uvm_test_top.c1.c3 [REPORT] comp3 report phase exited
UVM_INFO phase_order_ref.sv(89) @ 2000000: uvm_test_top.c1 [REPORT] comp1 report phase entered
UVM_INFO phase_order_ref.sv(90) @ 2000000: uvm_test_top.c1 [REPORT] comp1 report phase exited
UVM_INFO phase_order_ref.sv(121) @ 2000000: uvm_test_top [REPORT] phase_order_test report phase entered
UVM_INFO phase_order_ref.sv(122) @ 2000000: uvm_test_top [REPORT] phase_order_test report phase exited
UVM_INFO /opt/synopsys/vcs/vcs-mx/O-2018.09-SP2/etc/uvm-1.2/base/uvm_report_catcher.svh(705) @ 2000000: reporter [UVM/REPORT/CATCHER] 
*/
```
**注意run_phase和12个小phase尽量不要同时使用**

## uvm_config
```verilog
interface uvm_config_if;
  logic [31:0] addr;
  logic [31:0] data;
  logic [ 1:0] op;
endinterface

package uvm_config_pkg;
  import uvm_pkg::*;
  `include "uvm_macros.svh"
  
  class config_obj extends uvm_object;
    int comp1_var;
    int comp2_var;
    `uvm_object_utils(config_obj)
    function new(string name = "config_obj");
      super.new(name);
      `uvm_info("CREATE", $sformatf("config_obj type [%s] created", name), UVM_LOW)
    endfunction
  endclass
  
  class comp2 extends uvm_component;
    int var2;
    virtual uvm_config_if vif;  
    config_obj cfg; 
    `uvm_component_utils(comp2)
    function new(string name = "comp2", uvm_component parent = null);
      super.new(name, parent);
      var2 = 200;
      `uvm_info("CREATE", $sformatf("unit type [%s] created", name), UVM_LOW)
    endfunction
    function void build_phase(uvm_phase phase);
      super.build_phase(phase);
      `uvm_info("BUILD", "comp2 build phase entered", UVM_LOW)
      if(!uvm_config_db#(virtual uvm_config_if)::get(this, "", "vif", vif))
        `uvm_error("GETVIF", "no virtual interface is assigned")
        
      `uvm_info("GETINT", $sformatf("before config get, var2 = %0d", var2), UVM_LOW)
      uvm_config_db#(int)::get(this, "", "var2", var2);
      `uvm_info("GETINT", $sformatf("after config get, var2 = %0d", var2), UVM_LOW)
      
      uvm_config_db#(config_obj)::get(this, "", "cfg", cfg);
      `uvm_info("GETOBJ", $sformatf("after config get, cfg.comp2_var = %0d", cfg.comp2_var), UVM_LOW)     
      
      `uvm_info("BUILD", "comp2 build phase exited", UVM_LOW)
    endfunction
  endclass

  class comp1 extends uvm_component;
    int var1;
    comp2 c2;
    config_obj cfg; 
    virtual uvm_config_if vif;
    `uvm_component_utils(comp1)
    function new(string name = "comp1", uvm_component parent = null);
      super.new(name, parent);
      var1 = 100;
      `uvm_info("CREATE", $sformatf("unit type [%s] created", name), UVM_LOW)
    endfunction
    function void build_phase(uvm_phase phase);
      super.build_phase(phase);
      `uvm_info("BUILD", "comp1 build phase entered", UVM_LOW)
      if(!uvm_config_db#(virtual uvm_config_if)::get(this, "", "vif", vif))
        `uvm_error("GETVIF", "no virtual interface is assigned")
        
      `uvm_info("GETINT", $sformatf("before config get, var1 = %0d", var1), UVM_LOW)
      uvm_config_db#(int)::get(this, "", "var1", var1);
      `uvm_info("GETINT", $sformatf("after config get, var1 = %0d", var1), UVM_LOW)
      
      uvm_config_db#(config_obj)::get(this, "", "cfg", cfg);
      `uvm_info("GETOBJ", $sformatf("after config get, cfg.comp1_var = %0d", cfg.comp1_var), UVM_LOW)
      
      c2 = comp2::type_id::create("c2", this);
      `uvm_info("BUILD", "comp1 build phase exited", UVM_LOW)
    endfunction
  endclass

  class uvm_config_test extends uvm_test;
    comp1 c1;
    config_obj cfg;
    `uvm_component_utils(uvm_config_test)
    function new(string name = "uvm_config_test", uvm_component parent = null);
      super.new(name, parent);
    endfunction
    function void build_phase(uvm_phase phase);
      super.build_phase(phase);
      `uvm_info("BUILD", "uvm_config_test build phase entered", UVM_LOW)
      
      cfg = config_obj::type_id::create("cfg");
      cfg.comp1_var = 100;
      cfg.comp2_var = 200;
      uvm_config_db#(config_obj)::set(this, "*", "cfg", cfg);
      
      uvm_config_db#(int)::set(this, "c1", "var1", 10);
      uvm_config_db#(int)::set(this, "c1.c2", "var2", 20);
      
      c1 = comp1::type_id::create("c1", this);
      `uvm_info("BUILD", "uvm_config_test build phase exited", UVM_LOW)
    endfunction
    task run_phase(uvm_phase phase);
      super.run_phase(phase);
      `uvm_info("RUN", "uvm_config_test run phase entered", UVM_LOW)
      phase.raise_objection(this);
      #1us;
      phase.drop_objection(this);
      `uvm_info("RUN", "uvm_config_test run phase exited", UVM_LOW)
    endtask
  endclass
endpackage

module uvm_config_ref;

  import uvm_pkg::*;
  `include "uvm_macros.svh"
  import uvm_config_pkg::*;
  
  uvm_config_if if0();

  initial begin
    uvm_config_db#(virtual uvm_config_if)::set(uvm_root::get(), "uvm_test_top.*", "vif", if0);
    run_test(""); // empty test name
  end

endmodule
```