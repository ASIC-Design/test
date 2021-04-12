# test
low power design

1. one hot or differential code style ( counter 尽量使用独热码，格雷码）
2. In MUX， high toggle signals use less chain（高翻转率的信号接到mux的级联最小的端， 通常是cell级别）
3. signal gating， 在decoder的case中添加enbale，减少翻转
4. bus上翻转bits达到一半时，损耗最大，超过一半后变小
5. 
