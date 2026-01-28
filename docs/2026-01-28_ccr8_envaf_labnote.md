지금까지 한 걸 “공적인 실험 기록” 느낌으로 남길 수 있게, 아래 내용을 그대로 복붙해서 쓰면 된다.  
파일 맨 위부터 끝까지 통째로 넣어줘.

***

# CCR8–Envafolimab CDR3 설계 실험 기록

## 1. 실험 개요

- 일자: 2026-01-28  
- 담당자: 윤현주
- 프로젝트: CCR8 상부(ECL 및 상부 TM)를 타겟으로 하는 Envafolimab 기반 나노바디 설계  
- 목적:  
  - CCR8 리간드 결합 포켓 상부를 차폐(blocking)하는 Envafolimab 유래 나노바디를 설계.  
  - Envafolimab CDR-H3 구간을 구조·서열 재설계를 통해 CCR8 ECL 상부 포켓을 충전하는 형태로 최적화.  

## 2. 사용 데이터 및 구조 정보

### 2.1 CCR8 수용체

- PDB ID: 8TLM (CCR8–Gi 복합체 구조)  
- 사용 체인: CCR8 체인 C  
- 본 실험에서 사용한 상부 도메인(ECL-only) PDB:  
  - 파일 경로: `data/reference/ccr8_ecl_only.pdb`  
  - 포함 residue (체인 C 기준):  
    - 80–227  
    - 237–265  
    - 273–314  
  - 제거한 영역: ICL 전반, 하부 TM, Helix 8 등 세포질 측 영역.  

### 2.2 Envafolimab VHH (나노바디)

- 출처: Envafolimab VHH Alphafold 예측 모델 (로컬에서 mmCIF → PyMOL로 PDB 변환 후 사용). [etloveguitar.tistory](https://etloveguitar.tistory.com/20)
- 파일 경로: `data/reference/envafolimab_vhh.pdb` (PyMOL export 후 정리본 사용).  
- 체인 및 residue 정의 (PyMOL에서 확인):  
  - 전체 VHH 길이: residue 1–128  
  - IMGT 기준 CDR-H3 (CDR3) 구간: residue 98–117  
  - 본 실험에서 체인 기호는 D 체인으로 설정.  

### 2.3 복합체 입력 PDB

- 목적: CCR8 상부(ECL-only)와 Envafolimab VHH를 포함하는 복합체를 RFdiffusion 입력으로 사용.  
- 최종 파일: `data/reference/ccr8_ecl_envafolimab.pdb`  
- 체인 구성:  
  - 체인 C: CCR8 ECL-only (`ccr8_ecl_only.pdb`)  
  - 체인 D: Envafolimab VHH (residue 1–128, CDR-H3 = 98–117)  

## 3. 구조 전처리 및 PDB 생성 절차

### 3.1 CCR8 상부 도메인 추출

- 원본 구조: `data/reference/8TLM.pdb`  
- 수행 목적: RFdiffusion에 상부 도메인만 입력하여 계산 효율 및 타겟 포켓 명확화.  
- 처리 방법(요약):  
  - 체인 C 중 residue 80–227, 237–265, 273–314만 남기고 나머지 제거.  
  - ICL/하부 TM/Helix 8 영역 삭제 후 `TER`, `END` 레코드 추가.  

예시 명령(레시피용):

```bash
cd ~/hj/ccr8_project

grep '^ATOM' data/reference/8TLM.pdb \
 | awk '$5=="C" && $6>=80 && $6<=227 || $5=="C" && $6>=237 && $6<=265 || $5=="C" && $6>=273 && $6<=314' \
 > data/reference/ccr8_ecl_only.pdb
echo "TER" >> data/reference/ccr8_ecl_only.pdb
echo "END" >> data/reference/ccr8_ecl_only.pdb
```

### 3.2 Envafolimab VHH 체인 및 번호 정리

- 목적: RFdiffusion 입력에서 일관된 체인명(D)과 residue index를 사용하기 위함.  
- 처리 개요:  
  - PyMOL에서 불필요한 이종분자 제거 및 단일 체인으로 정리 후 PDB export.  
  - Bash/awk를 사용하여 체인명을 D로 통일한 파일(`envafolimab_vhh_D.pdb`) 생성.  

예시 명령:

```bash
awk 'substr($1,1,4)=="ATOM"{
  printf("%-6s%5d %-4s%1s%-3s %1s%4d    %8.3f%8.3f%8.3f %5.2f %6.2f           %2s\n",
         "ATOM",NR,$3,"",$4,"D",$6,$7,$8,$9,$10,$11,$12); next}1' \
  data/reference/envafolimab_vhh_clean.pdb \
  > data/reference/envafolimab_vhh_D.pdb
```

### 3.3 CCR8–Envafolimab 복합체 생성

- 목적: RFdiffusion binder design 모드 입력으로 사용할 복합체 PDB 생성.  
- 처리: CCR8 ECL-only(PDB)와 Envafolimab VHH(PDB)를 단순 병합 후 `TER`, `END` 추가.  

```bash
cat data/reference/ccr8_ecl_only.pdb data/reference/envafolimab_vhh_D.pdb \
  > data/reference/ccr8_ecl_envafolimab.pdb
echo "TER" >> data/reference/ccr8_ecl_envafolimab.pdb
echo "END" >> data/reference/ccr8_ecl_envafolimab.pdb
```

## 4. RFdiffusion 설정 및 실행 내역

### 4.1 공통 설정

- 모델 모드: protein–protein binder design (SelfConditioning)  
- 입력 PDB: `data/reference/ccr8_ecl_envafolimab.pdb`  
- 출력 경로 prefix: `data/processed/envaf_ccr8_cdr3`  
- 설계 개수: 시범 런 3개 (`inference.num_designs=3`)  

### 4.2 Contig 및 마스킹 설정

- receptor (CCR8, 체인 C) contig:  

  - `C80-227/C237-265/C273-314`  
  - 하나의 연속된 receptor 체인으로 취급하되, 실제 PDB 상에서는 ECL/상부 TM fragment로 구성.  

- binder (Envafolimab, 체인 D) contig:  

  - `D1-128` (VHH 전체 길이)  

- 설계(inpaint) 대상 구간: Envafolimab CDR-H3  
  - 체인 D, residue 98–117  
  - 서열/구조 마스킹을 모두 적용하여 해당 구간을 새로 설계하도록 설정.  

- hotspot residue (CCR8 ECL 상호작용 잔기):  
  - 체인 C, residue 97, 106, 183, 283  
  - 바인더가 이 부위와 상호작용하도록 유도.  

### 4.3 실행에 사용한 명령

```bash
python external_tools/RFdiffusion/scripts/run_inference.py \
  inference.input_pdb=data/reference/ccr8_ecl_envafolimab.pdb \
  inference.num_designs=3 \
  inference.output_prefix=data/processed/envaf_ccr8_cdr3 \
  'contigmap.contigs=[C80-227/C237-265/C273-314/0 D1-128]' \
  "ppi.hotspot_res=[C97,C106,C183,C283]" \
  "contigmap.inpaint_seq=['D98','D99','D100','D101','D102','D103','D104','D105','D106','D107','D108','D109','D110','D111','D112','D113','D114','D115','D116','D117']" \
  "contigmap.inpaint_str=['D98','D99','D100','D101','D102','D103','D104','D105','D106','D107','D108','D109','D110','D111','D112','D113','D114','D115','D116','D117']"
```

### 4.4 결과 요약

- 생성된 출력 파일:  
  - `data/processed/envaf_ccr8_cdr3_*.pdb` (총 3개)  
- 관찰 사항(시각적 검사 기준):  
  - Envafolimab 체인 D 프레임워크(residue 1–97, 118–128)는 원형 구조를 유지.  
  - CDR-H3 (D98–117)은 새로 설계된 서열·구조로 치환되며, CCR8 ECL 상부 포켓을 향해 돌출되는 형태를 보임(자세한 품질 평가는 후속 분석 필요).  

## 5. 후속 계획 및 TODO

1. CDR-H3 형상 제약 강화  
   - inpaint_str 옵션을 활용하여 CDR3 구간의 2차 구조(β-turn, loop 등) 패턴을 명시적으로 부여하는 조건 실험.  
2. 포켓 내부 차폐(blocking) 강화  
   - CCR8 ECL 포켓 내부 특정 잔기를 anchor로 선정하고, 해당 부위와의 접촉을 극대화하는 가이드 포텐셜 사용 검토.  
3. 디자인 품질 평가  
   - 각 디자인에 대해 (1) CCR8 접촉면적, (2) CDR3 packing, (3) 예상 developability(표면 전하, 히드로포빅 패치 등) 정량 평가 예정.  
4. 재현성 확보  
   - 동일 조건으로 seed 변경 실험 계획,  
   - 본 문서에 seed, 랜덤 초기 조건 등 메타데이터 추가 기록 예정.  

