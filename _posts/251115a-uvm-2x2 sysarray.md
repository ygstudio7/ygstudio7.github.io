참고: https://chatgpt.com/c/69194e42-da4c-8328-b46b-3bc72ec110c6



# 2x2 systolic array







원하면:

✨ **matmul 여러 개를 bank 0/1/0/1 순서로 연속 실행하는 TB**
 ✨ **pipeline overlap 3~4 matmul 버전**
 ✨ **N×N 확장 자동화된 generator**





# 제한 사항

NUM_BANKS = 1,2,4 : 2의 배수 -> 확인 방법

[제한점] 

1. b load는 1회만 되도록 간단히 만들었다. 이유는 b_col_loader의 state가 load가 완료되면 ST_DONE에 가있기 때문에, start를 해도 다시 시작하지 않는다
2. bank수는 MAT개수와 같다. -> 최적화 X
3. 



[ ] DATA_W -> DW

[ ] ACC_W = DATA_W*2+1?

[ ] NUM_ -> N_: NUM_MATMULS -. N_MATMULS, NUM_BANKS -> N_BANKS

REVERSE=1				REVERSE=0

ST_LOAD_ROWS <> ST_SEND_ROWS			--> ST_FEED_ROWS

row 역순 --	==0		row ++ == M-1				--> rows ++

ST_B2B_DELAY		ST_M2M_DELAY 				-> ST_M2M_DELAY

w_load_en					a_valid_col					-> m_valid

w_top_in						a_left_in_col				-> m_out

w_load_bank				bank_idx_col				-> m_bank_idx