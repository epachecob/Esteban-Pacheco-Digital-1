module multiplicador_binario (
    input clk,
    input reset,
    input start,
    input [7:0] A, // Multiplicando
    input [7:0] B, // Multiplicador
    output reg [15:0] resultado,
    output reg ready
    );

    // Estados de la FSM
    parameter IDLE      = 3'b000;
    parameter INICIO    = 3'b001;
    parameter MULT      = 3'b010;
    parameter DESPLAZAR = 3'b011;
    parameter SUMAR     = 3'b100;
    parameter FIN       = 3'b101;

    reg [2:0] estado;
    reg [15:0] multiplicando_reg;
    reg [7:0] multiplicador_reg;
    reg [3:0] contador;

    always @(posedge clk or posedge reset) begin
        if (reset) begin
            estado <= IDLE;
            ready <= 0;
            resultado <= 0;
        end else begin
            case (estado)
                IDLE: begin
                    ready <= 0;
                    if (start) estado <= INICIO;
                end

                INICIO: begin
                    // "Tomar los números A y B"
                    // Cargamos A en un registro de 16 bits para el desplazamiento
                    multiplicando_reg <= {8'b0, A}; 
                    multiplicador_reg <= B;
                    resultado <= 16'b0;
                    contador <= 0;
                    estado <= MULT;
                end

                MULT: begin
                    // "Se multiplica el LSB del multiplicador por cada bit del multiplicando"
                    // Si el LSB es 1, sumamos el multiplicando actual al resultado
                    if (multiplicador_reg[0]) begin
                        resultado <= resultado + multiplicando_reg;
                    end
                    estado <= DESPLAZAR;
                end

                DESPLAZAR: begin
                    // "El resultado de la multiplicación tendrá un corrimiento a la izquierda
                    multiplicando_reg <= multiplicando_reg << 1;
                    
                    // "Se realiza un corrimiento a la derecha del multiplicador"
                    multiplicador_reg <= multiplicador_reg >> 1;
                    
                    contador <= contador + 1;
                    estado <= (contador < 7) ? MULT : FIN; 
                end

                FIN: begin
                    ready <= 1;
                    if (!start) estado <= IDLE;
                end
            endcase
        end
    end
endmodule
