Code for The Vhdl Vending Machine:
LIBRARY ieee;
USE ieee.std_logic_1164.all;
USE ieee.numeric_std.all;

entity vendingMachine is
  port(clock : in STD_LOGIC;
		  -- Switch to swap between input meaning counting money and then selecting a drink
		  SS : in STD_LOGIC;
        SW   : in  std_logic_vector(1 downto 0);
		  -- key(0) = Counter key(1) = reset key   key(2) = show selection 
        KEY  : in  std_logic_vector(2 downto 0);
        HEX0, HEX1, HEX2, HEX3, HEX4, HEX5: out STD_LOGIC_VECTOR(6 DOWNTO 0));
		  
end vendingMachine;

architecture Behavioral of vendingMachine is
type selectionType is (stateOne, stateTwo, stateThree, stateFour, stateMoney, stateVend);
signal selection : selectionType;
signal selectionCount : INTEGER := 1;
signal buttonCount : INTEGER := 1;
signal delayCount : INTEGER := 1;

signal vendDelayBegin : STD_LOGIC;
signal vendDelayCount : integer := 1;
signal vendDelay : integer := 100000000;

signal itemCost : integer := 75;
signal balance : integer := 0;
signal coin    : integer := 0;

function numberToVector(number : INTEGER; hex1Or0 : STD_LOGIC) return STD_LOGIC_VECTOR is 
	variable result : STD_LOGIC_VECTOR(6 DOWNTO 0);
	begin
	if hex1Or0 = '0' then
		case (number mod 10) is
			when 0 =>
				return "0000001";
			when 1 =>
				return "1001111";
			when 2 =>
				return "0010010";
			when 3 =>
				return "0000110";
			when 4 =>
				return "1001100";
			when 5 =>
				return "0100100";
			when 6 =>
				return "0100000";
			when 7 => 
				return "0001111";
			when 8 =>
				return "0000000";
			when others =>
				return "0000100";
		end case;
				
	else
		case (number / 10) is
			when 0 =>
				return "0000001";
			when 1 =>
				return "1001111";
			when 2 =>
				return "0010010";
			when 3 =>
				return "0000110";
			when 4 =>
				return "1001100";
			when 5 =>
				return "0100100";
			when 6 =>
				return "0100000";
			when 7 => 
				return "0001111";
			when 8 =>
				return "0000000";
			when others =>
				return "0000100";
		end case;
		
	end if;
					 
	return result;
	
	end function;
	
begin

	PROCESS(clock, key(1))
		BEGIN
		
		if rising_edge(clock) then
			if key(1) = '0' then 
				selectionCount <= 0;
				balance <= 0;
				buttonCount <= 0;
				selection <= stateMoney;
				
			elsif SS = '1' and balance = 75 then
				if (vendDelayBegin = '0') then
					case SW(1 downto 0) is
							when "00" =>
								 SELECTION <= STATEONE;
							when "01" =>
								 SELECTION <= STATETWO;
							when "10" =>
								 SELECTION <= STATETHREE;
							when "11" =>
								 SELECTION <= STATEFOUR;
						end case;
				end if;
						
				if key(0) = '0' then
					vendDelayBegin <= '1';
					selection <= stateVend;
				end if;
				
				if vendDelayBegin = '1' then
					vendDelaycount <= vendDelayCount + 1;
					if (vendDelayCount = vendDelay) then
						selection <= stateMoney;
						balance <= 0;
						buttonCount <= 0;
						vendDelayBegin <= '0';
						vendDelayCount <= 0;
					end if;
				end if;
				
			elsif SS = '0' then
				if key(2) = '0' then 
					selectionCount <= selectionCount + 1;
							if (selectionCount = 90000021) then
								case selection is
									when stateOne => 
										selection <= stateTwo;
									when stateTwo => 
										selection <= stateThree;
									when stateThree => 
										selection <= stateFour;
									when stateFour => 
										selection <= stateOne;
									when stateMoney =>
										selection <= stateOne;
									when others =>
										null;
								end case;
								selectionCount <= 0;
							end if;
				
				elsif key(2) = '1' then
					selection <= stateMoney;
					case SW(1 downto 0) is
							when "00" =>
								 coin <= 0;
							when "01" =>
								 coin <= 5;
							when "10" =>
								 coin <= 10;
							when "11" =>
								 coin <= 25;
							when others =>
								 coin <= 0;
						end case;
						
					if key(0) = '0' then
							buttonCount <= buttonCount + 1;
							if (buttonCount = 11750000) then
								if (balance + coin) >= 75 then
									balance <= 75;
								else	
									balance <= balance + coin;
									buttonCount <= 0;
								end if;
							end if;
					end if;
				end if;
			end if;
		end if;
		
	END PROCESS;
	 
	 HEX0 <= "1111111" when selection = stateVend else
				numberToVector(itemCost, '0') when selection = stateOne or selection = stateTwo or selection = stateThree or selection = stateFour else
				numberToVector(balance, '0') when selection = stateMoney;
				
	 HEX1 <= "1111111" when selection = stateVend else
				numberToVector(itemCost, '1') when selection = stateOne or selection = stateTwo or selection = stateThree or selection = stateFour else
				numberToVector(balance, '1') when selection = stateMoney;
				
	 HEX2 <= "0001000" when selection = stateOne else
				"0110001" when selection = stateTwo else
				"0100100" when selection = stateThree else
				"1111111" when selection = stateFour or selection = stateMoney else
				"1000010" when selection = statevend else
				null;
				
	 HEX3 <= "1100000" when selection = stateOne else
			  "0000001" when selection = stateTwo else
			  "0011000" when selection = stateThree else
			  "0000100" when selection = stateFour else
			  "1111111" when selection = stateMoney else
			  "1101010" when selection = statevend else
			  null;
			  
	 HEX4 <= "0000001" when selection = stateOne else
			   "1001000" when selection = stateTwo else
			   "1000001" when selection = stateThree else
			   "0000100" when selection = stateFour else
			   "1111111" when selection = stateMoney else
				"0110000" when selection = statevend else
				null;
		
	 HEX5 <= "0110001" when selection = stateOne or selection = stateTwo else
				"0000000" when selection = stateThree else
				"0110000" when selection = stateFour else
				"1111111" when selection = stateMoney else
				"1000001" when selection = statevend else
				null;
				
end Behavioral;
