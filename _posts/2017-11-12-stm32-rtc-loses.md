---
#layout: post
title:  "STM32L4 RTC Reset 후 몇 millisecond 빠지는 문제"
date:   2017-11-12 16:16:01 -0900
categories: jekyll update
---

# {{page.title}}

RTC Clock을 LSE로 설정한 경우 Shutdown 상태에서도 RTC를 살려놓고, Auto Wakeup Timer를 이용해서 특정 시간마다 Wakeup 시켜서 필요한 작업을 수행하도록 할 수 있다.

센서에서 수집된 값을 DB에 저장하다보니 MariaDB에서 CURRENT_TIMESTAMP(6)로 수집한 Timestamp가 점점 늘어지는 문제가 발견되서 구글링을 하다보니 이미 같은 문제를 해결한 사례가 있었다.[링크](https://community.st.com/thread/41058-stm32-rtc-loses-one-second-after-each-reset)

문제의 핵심은 LSE를 설정하는 아래의 코드는 전원이 올라갈때 최초 1회만 실행되면 Reset을 해도 다시 수행할 필요가 없는데 Reset이 될때마다 실행되면서 LSE가 잠깐 멈춰게 되는 듯합니다.
```c
RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_LSE|RCC_OSCILLATORTYPE_MSI;
RCC_OscInitStruct.LSEState = RCC_LSE_ON;
RCC_OscInitStruct.MSIState = RCC_MSI_ON;
RCC_OscInitStruct.MSICalibrationValue = 0;
RCC_OscInitStruct.MSIClockRange = RCC_MSIRANGE_6;
RCC_OscInitStruct.PLL.PLLState = RCC_PLL_NONE;
if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK)
{
  _Error_Handler(__FILE__, __LINE__);
}
```

MX로 생성된 `MX_RTC_Init()` 함수를 보면 RTC가 성공 적으로 초기화될 경우 `RTC_BKP_DR0` register에 "`0x32F2`"를 저장하고, 다음에 실행될 때는 이 레지스터에 저장된 값을 이용해서 최초 전원이 켜진건지, Reset이 된건지를 알 수 있다.

```c
if(HAL_RTCEx_BKUPRead(&hrtc, RTC_BKP_DR0) != 0x32F2) { // Power Up 상황인가? Reset 상황인가?
...
  HAL_RTCEx_BKUPWrite(&hrtc,RTC_BKP_DR0,0x32F2);       // 최초 Power Up 시 RTC_BKP_DR0에 저장
...
}
```

`RTC_BKP_DR0` register의 값을 이용하면 LSE를 초기화하는 코드도 최초 전원이 켜질때만 실행 되도록 바꿀 수 있다.
```c
hrtc.instance = RTC;

RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_MSI;

// RTC가 이미 설정 되있지 않는 경우만 RCC_OSCILLATORTYPE_LSE를 초기화
if(HAL_RTCEx_BKUPRead(&hrtc, RTC_BKP_DR0) != 0x32F2) {
  RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_LSE|RCC_OSCILLATORTYPE_MSI;
  RCC_OscInitStruct.LSEState = RCC_LSE_ON;
}

RCC_OscInitStruct.MSIState = RCC_MSI_ON;
RCC_OscInitStruct.MSICalibrationValue = 0;
RCC_OscInitStruct.MSIClockRange = RCC_MSIRANGE_6;
RCC_OscInitStruct.PLL.PLLState = RCC_PLL_NONE;
if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK)
{
  _Error_Handler(__FILE__, __LINE__);
}

```

추가로 `MX_RTC_Init()`함수도 많이 수정을 해줘야된다. `if(HAL_RTCEx_BKUPRead(&hrtc, RTC_BKP_DR0) != 0x32F2)` 이 if문 밖에 있는 대부분의 내용을 안쪽으로 넣으면 된다.
```c
static void MX_RTC_Init(void)
{

  RTC_TimeTypeDef sTime;
  RTC_DateTypeDef sDate;

  hrtc.Instance = RTC;
  /**Initialize RTC and set the Time and Date 
  */
  if(HAL_RTCEx_BKUPRead(&hrtc, RTC_BKP_DR0) != 0x32F2){
    /**Initialize RTC Only 
    */
    hrtc.Init.HourFormat = RTC_HOURFORMAT_24;
    hrtc.Init.AsynchPrediv = 127;
    hrtc.Init.SynchPrediv = 255;
    hrtc.Init.OutPut = RTC_OUTPUT_DISABLE;
    hrtc.Init.OutPutRemap = RTC_OUTPUT_REMAP_NONE;
    hrtc.Init.OutPutPolarity = RTC_OUTPUT_POLARITY_HIGH;
    hrtc.Init.OutPutType = RTC_OUTPUT_TYPE_OPENDRAIN;

    if (HAL_RTC_Init(&hrtc) != HAL_OK)
    {
      _Error_Handler(__FILE__, __LINE__);
    }

    sTime.Hours = 0x0;
    sTime.Minutes = 0x0;
    sTime.Seconds = 0x0;
    sTime.DayLightSaving = RTC_DAYLIGHTSAVING_NONE;
    sTime.StoreOperation = RTC_STOREOPERATION_RESET;
    if (HAL_RTC_SetTime(&hrtc, &sTime, RTC_FORMAT_BCD) != HAL_OK)
    {
      _Error_Handler(__FILE__, __LINE__);
    }

    sDate.WeekDay = RTC_WEEKDAY_MONDAY;
    sDate.Month = RTC_MONTH_JANUARY;
    sDate.Date = 0x1;
    sDate.Year = 0x0;

    if (HAL_RTC_SetDate(&hrtc, &sDate, RTC_FORMAT_BCD) != HAL_OK)
    {
      _Error_Handler(__FILE__, __LINE__);
    }

    /**Enable the WakeUp 
    */
    if (HAL_RTCEx_SetWakeUpTimer_IT(&hrtc, 119, RTC_WAKEUPCLOCK_CK_SPRE_16BITS) != HAL_OK)
    {
      _Error_Handler(__FILE__, __LINE__);
    }

    HAL_RTCEx_BKUPWrite(&hrtc,RTC_BKP_DR0,0x32F2);
  }

  /* Clear the EXTI's line Flag for RTC WakeUpTimer */
  __HAL_RTC_WAKEUPTIMER_EXTI_CLEAR_FLAG();
  
  /* Get the pending status of the WAKEUPTIMER Interrupt */
  if(__HAL_RTC_WAKEUPTIMER_GET_FLAG(&hrtc, RTC_FLAG_WUTF) != RESET)
  {   
    /* Clear the WAKEUPTIMER interrupt pending bit */
    __HAL_RTC_WAKEUPTIMER_CLEAR_FLAG(&hrtc, RTC_FLAG_WUTF);

    /* WAKEUPTIMER callback */ 
    HAL_RTCEx_WakeUpTimerEventCallback(&hrtc);
  }
}
```

이렇게 했더니 이제는 오히려 시간이 빨라졌지만 그건 LSE의 오차라고 생각함.
