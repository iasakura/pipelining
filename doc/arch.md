# コード生成と生成されるアーキテクチャ

- gated SSA -> CDFG -> verilog

## トップジュール

- clk, rst_n, start, finish, args, ret (tuple)

args は wire

## CFG

- 状態機械になる
- 終了状態を表す特殊な状態 FIN を追加

- rst_n で初期化
- en で開始、fin が来ると状態遷移
    - fin を観測した clock で次の BB の初期化もやる

### I/F
- BB_en: out
- BB_done: in

### 内部状態
- enum state { BB1, BB2, ... }
    - cur_state/prev_state
        - prev_state は eta ノードのために必要

## BB
以下の I/F を持つモジュールになる

- en
- done

レイテンシは可変とする


### External module

```rust
resource FIFO_i32 {
    read() -> (i32) @ Variable,
}

resource ADDR {
    call (int(32), int(32)) -> int(32) @ Combinatorial
}

resource BRAM_2P {
    // Latency=II=1, 
    read: (int(32)) -> int(32) @ Fixed(1, 1) 
    // Latency=2, II=1
    write: (int(32), int(32)) -> int(32) @ Fixed(2, 1)
}

cdfg sample {
    starts INIT;
    // Arguments are either scalar or port
    params (n: int(32), fifo: FIFO_i32);
    returns ret;
    resources {
        // internal module instantiation
        module add_0 : ADDR
        module bram_0 : BRAM_2P 
    }
    cdfg {
        INIT: {

        } exit(jc(test0, LOOP, EXIT))
        LOOP(INIT) {
            sum = mu(0, sum);
            sum_next = call add_0.call(i, sum)
            v = call bram0.read(...)
            w = call bram0.write(...) depends [v]
        } exit(jmp(EXIT))
        EXIT(INIT, LOOP) {

        } exit(ret)
    }
```

### Sum of array
 
```rust
module sum_of_array {
    starts INIT,
    params (n: int(32), arr: BRAM_2P),
    returns ret,
    resources {

    },
}
```