void Usart_Init(void)
{
	
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1 | RCC_APB2Periph_GPIOA | RCC_APB2Periph_AFIO,ENABLE);
	
	
	GPIO_InitTypeDef GPIO_InitStruct; 

	
	GPIO_InitStruct.GPIO_Pin  = GPIO_Pin_9 ;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AF_PP;
	GPIO_Init(GPIOA,&GPIO_InitStruct);
	
	GPIO_InitStruct.GPIO_Pin  = GPIO_Pin_10 ;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IN_FLOATING;
	GPIO_Init(GPIOA,&GPIO_InitStruct);
	



	NVIC_InitTypeDef NVIC_InitStruct;
	NVIC_PriorityGroupConfig(NVIC_PriorityGroup_1);

	NVIC_InitStruct.NVIC_IRQChannel = USART1_IRQn;
	NVIC_InitStruct.NVIC_IRQChannelPreemptionPriority = 1;
	NVIC_InitStruct.NVIC_IRQChannelSubPriority = 4;
	NVIC_InitStruct.NVIC_IRQChannelCmd = ENABLE;
	NVIC_Init(&NVIC_InitStruct);
	
	
	
	USART_InitTypeDef USART_InitStruct;
	
	USART_InitStruct.USART_BaudRate = 9600;
	USART_InitStruct.USART_WordLength = USART_WordLength_8b;
	USART_InitStruct.USART_StopBits = USART_StopBits_1;
	USART_InitStruct.USART_HardwareFlowControl = USART_HardwareFlowControl_None;
	USART_InitStruct.USART_Parity = USART_Parity_No ;
	USART_InitStruct.USART_Mode =  USART_Mode_Tx | USART_Mode_Rx;
	
	USART_Init(USART1, &USART_InitStruct);
	
	
	USART_ITConfig(USART1,USART_IT_TXE,DISABLE);
	USART_ITConfig(USART1,USART_IT_RXNE,ENABLE);
	
	USART_Cmd(USART1,ENABLE);
	
	
		
}

void USART1_IRQHandler(void)
{	
		if(USART_GetFlagStatus(USART1,USART_FLAG_RXNE) == SET)
		{
			if (flag == 0)
			{
				USART_ClearFlag(USART1,USART_FLAG_RXNE);
			
					Rxbuffer[Rxcounter] = USART_ReceiveData(USART1);
					Rxcounter++;
					while(USART_GetFlagStatus(USART1,USART_FLAG_RXNE) == RESET);
				
			}
			if (Rxcounter == NDataToRead )
			{	
				USART_ITConfig(USART1,USART_IT_RXNE,DISABLE);					
				USART_ITConfig(USART1,USART_IT_TXE,ENABLE);		
				flag = 1;
				Rxcounter = 0;
			}
		}
	
		if(USART_GetFlagStatus(USART1,USART_FLAG_TXE) == SET)
		{
			if (flag == 1)
			{			
				USART_ClearFlag(USART1,USART_FLAG_TXE);
				USART_SendData(USART1,Rxbuffer[Rxcounter1]);
				Rxcounter1++;
				while(USART_GetFlagStatus(USART1,USART_FLAG_TXE) != SET);
				if (Rxcounter1 == NDataToRead)
				{
					flag = 0;
					
					USART_ITConfig(USART1,USART_IT_TXE,DISABLE);
					USART_ITConfig(USART1,USART_IT_RXNE,ENABLE);	
					Rxcounter1 = 0;
					for (i = 0;i < 10;i++)
					{
						Rxbuffer[i] = '\0';
					}
				}
			}
		
		}		
