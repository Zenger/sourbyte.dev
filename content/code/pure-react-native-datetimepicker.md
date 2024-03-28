+++
title = "Pure React Native Datetimepicker"
date = "2023-07-02 03:38:00"
+++

## React Native Date and Time Picker

If you're on this page that means like myself you were overwhelmed by the sheer volume of crazy date and time pickers that simply dont work or are so goofy that I cant understand why they even exist? /s.

Some date pickers are very well done, see [UI Kitten Datepicker](https://akveo.github.io/react-native-ui-kitten/docs/components/datepicker/overview#datepicker) but it is missing a time picker, and to be perfectly honest if I've found this before I spent a whole day making a new component I would've used that and added a hour/minute input and called it a day.

NOTE: I leverage all date/time calculations to [momentjs](https://momentjs.com/) so it is required.

But I digress so here it is in all its glory.

<video width="100%" controls>
    <source src="/assets/videos/pure-react-native-datetime-picker.mp4" type="video/mp4">
</video>

Using it is fairly simple:

```typescript
<DateTimePicker
  label="Select Date"
  value={my_date ? moment(my_date).toDate() : new Date()}
  onDateTimeSelect={(date: Date) => {
    /* onChange({label: 'scheduled_at', value: date}) */
  }}
/>
```

and the code for it, there are are few constants for theming adjust as you see fit:

```typescript
import React, { useState } from "react";
import {
  Button,
  Modal,
  Pressable,
  ScrollView,
  StyleSheet,
  Text,
  TextInput,
  TextInputProps,
  View,
} from "react-native";
import moment, { Moment } from "moment";

const Theme = {
  colors: {
    background: "#191722",
    primary: "#fb8501",
    text: "#ffffff",
  },
};
const Locale = {
  weekdays: ["Sun", "Mon", "Tues", "Wed", "Thu", "Fri", "Sat"],
  months: [
    "January",
    "February",
    "March",
    "April",
    "May",
    "June",
    "July",
    "August",
    "September",
    "October",
    "November",
    "December",
  ],
};

const ColorScheme = {
  background: Theme.colors.background,
  labelColor: Theme.colors.primary,
  text: Theme.colors.text,
  borderColor: "#ffffff",
  borderWidth: StyleSheet.hairlineWidth,
  inactive: "#3d3d3d",
  today: "#fb8501",
  selectedBackground: "#fb8501",
  selectedText: "black",
  todayBackground: "rgba(251,133,1,0.2)",
  todayText: "#fb8501",
};

interface DateTimePickerProps {
  value: Date;
  label: string;
  onDateTimeSelect: (date: Date) => void;
}

interface Day {
  date: Moment;
  selected: boolean;
  disabled?: boolean;
}

const Input = (props: TextInputProps) => {
  return (
    <TextInput style={{ backgroundColor: ColorScheme.background }} {...props} />
  );
};

export const DateTimePicker: React.FC<DateTimePickerProps> = (props: any) => {
  const [open, setOpen] = useState<boolean>(false);
  const [date, setDate] = useState<Date>(props.value);

  const [monthSelectorVisible, setMonthSelectorVisible] =
    useState<boolean>(false);
  const [yearEntry, setYearEntry] = useState<string>(
    moment(date).format("YYYY")
  );

  const [selectedDateTime, setSelectedDateTime] = useState<Moment>(
    moment(date)
  );

  const [minuteEntry, setMinuteEntry] = useState<string>(
    moment(date).format("mm")
  );
  const [hourEntry, setHourEntry] = useState<string>(moment(date).format("hh"));

  const label = () => {
    return selectedDateTime.format("ddd, MMM DD, YYYY HH:mm");
  };

  const PressableFeedback = (props: any) => {
    return (
      <Pressable
        {...props}
        style={({ pressed }) => {
          return pressed
            ? [
                { opacity: 0.6, backgroundColor: "rgba(0,0,0,0.2)" },
                props.style,
              ]
            : props.style;
        }}
      >
        {props.children}
      </Pressable>
    );
  };

  const decreaseMonth = () => {
    setDate(moment(date).subtract(1, "month").toDate());
  };

  const increaseMonth = () => {
    setDate(moment(date).add(1, "month").toDate());
  };

  const advanceToMonth = (month: number) => {
    setDate(moment(date).month(month).toDate());
    setMonthSelectorVisible(false);
  };

  const advanceToYear = () => {
    let _year = Number(yearEntry);
    if (_year < 1966) {
      return;
    }
    if (_year > 3000) {
      return;
    }

    setDate(moment(date).year(_year).toDate());
  };

  const renderHeader = () => {
    return (
      <View style={{ flex: 1, flexDirection: "row" }}>
        <PressableFeedback
          style={{ padding: 5, flex: 1 }}
          onPress={decreaseMonth}
        >
          <Text style={{ color: ColorScheme.text, fontSize: 40 }}>{"<"}</Text>
        </PressableFeedback>
        <View style={{ marginTop: 8 }}>
          <PressableFeedback
            style={{ padding: 5, flex: 2 }}
            onPress={() => {
              setMonthSelectorVisible(!monthSelectorVisible);
            }}
          >
            <Text style={{ color: ColorScheme.text, fontSize: 36 }}>
              {moment(date).format("MMMM")}
            </Text>
          </PressableFeedback>
          {monthSelectorVisible && (
            <View
              style={{
                position: "absolute",
                top: 0,
                left: 0,
                flex: 1,
                backgroundColor: "white",
                width: "100%",
                zIndex: 10001000,
              }}
            >
              {Locale.months.map((month, index) => {
                return (
                  <PressableFeedback
                    onPress={() => {
                      advanceToMonth(index);
                    }}
                    key={index}
                    style={{
                      height: 40,
                      padding: 5,
                      borderBottomColor: ColorScheme.borderColor,
                      borderWidth: ColorScheme.borderWidth,
                    }}
                  >
                    <Text style={{ fontSize: 18 }}>{month}</Text>
                  </PressableFeedback>
                );
              })}
            </View>
          )}
        </View>
        <View style={{ padding: 5, flex: 2 }}>
          <Input
            value={yearEntry}
            defaultValue={yearEntry}
            onChangeText={(year) => setYearEntry(year)}
            onBlur={advanceToYear}
            editable
            style={{
              color: "white",
              fontSize: 36,
              borderBottomColor: ColorScheme.text,
              borderBottomWidth: StyleSheet.hairlineWidth,
            }}
            keyboardType="numeric"
          />
        </View>
        <PressableFeedback
          style={{ padding: 5, marginLeft: "auto" }}
          onPress={increaseMonth}
        >
          <Text style={{ color: ColorScheme.text, fontSize: 40 }}>{">"}</Text>
        </PressableFeedback>
      </View>
    );
  };

  const renderWeekdays = () => {
    return (
      <View style={{ flexDirection: "row" }}>
        {Locale.weekdays.map((day, index) => {
          return (
            <View
              key={index}
              style={{
                flex: 1,
                padding: 5,
                paddingVertical: 20,
                borderColor: ColorScheme.borderColor,
                borderWidth: ColorScheme.borderWidth,
              }}
            >
              <Text style={{ color: ColorScheme.text, textAlign: "center" }}>
                {day}
              </Text>
            </View>
          );
        })}
      </View>
    );
  };
  const selectDay = (date: Moment) => {
    date.minute(Number(minuteEntry));
    date.hour(Number(hourEntry));
    setSelectedDateTime(date);
  };

  const renderDays = () => {
    const startOfTheMonth = moment(date).startOf("month");
    const firstDayOfTheMonth = Number(startOfTheMonth.format("d"));
    const lastDayOfTheMonth = Number(moment(date).endOf("month").format("d"));

    const grid: Day[] = [];

    for (let i = 0; i < 42; i++) {
      if (i < firstDayOfTheMonth) {
        grid.push({
          date: moment(date)
            .startOf("month")
            .subtract(firstDayOfTheMonth - i, "days"),
          selected: false,
          disabled: false,
        });
      } else if (i === firstDayOfTheMonth) {
        grid.push({
          date: moment(date).startOf("month"),
          selected: false,
          disabled: false,
        });
      } else {
        grid.push({
          date: moment(date)
            .startOf("month")
            .add(i - firstDayOfTheMonth, "days"),
          selected: false,
          disabled: false,
        });
      }
    }

    let _return = [];
    for (let i = 0; i < 6; i++) {
      _return.push(
        <View key={i} style={{ flex: 1, flexDirection: "row" }}>
          {grid.slice(i * 7, i * 7 + 7).map((day, index) => {
            let isDaySelected =
              day.date.isSame(selectedDateTime, "day") &&
              day.date.isSame(selectedDateTime, "month") &&
              day.date.isSame(selectedDateTime, "year");
            let isToday =
              day.date.isSame(new Date(), "day") &&
              day.date.isSame(new Date(), "month") &&
              day.date.isSame(new Date(), "year");
            let backgroundStyle = {
              backgroundColor: isDaySelected
                ? ColorScheme.selectedBackground
                : isToday
                ? ColorScheme.todayBackground
                : "transparent",
            };
            let textStyle = {};

            return (
              <PressableFeedback
                onPress={() => {
                  selectDay(day.date);
                }}
                key={index}
                style={{
                  flex: 1,
                  paddingHorizontal: 4,
                  paddingVertical: 15,
                  borderColor: ColorScheme.borderColor,
                  borderWidth: ColorScheme.borderWidth,
                  backgroundColor: isDaySelected
                    ? ColorScheme.selectedBackground
                    : isToday
                    ? ColorScheme.todayBackground
                    : "transparent",
                }}
              >
                <Text
                  style={{
                    color: isDaySelected
                      ? ColorScheme.selectedText
                      : isToday
                      ? ColorScheme.todayText
                      : ColorScheme.text,
                    textAlign: "center",
                    fontWeight: isToday || isDaySelected ? "bold" : "normal",
                  }}
                >
                  {day.date.format("D")}
                </Text>
              </PressableFeedback>
            );
          })}
        </View>
      );
    }

    return _return;
  };

  return (
    <View>
      <Pressable
        onPress={() => {
          setOpen(!open);
        }}
        style={({ pressed }) => [
          pressed ? { opacity: 0.6, backgroundColor: "rgba(0,0,0,0.2)" } : {},
        ]}
      >
        <View
          style={{
            borderWidth: StyleSheet.hairlineWidth,
            borderColor: "#3d3d3d",
            paddingHorizontal: 10,
            paddingVertical: 10,
            flexDirection: "row",
          }}
        >
          <Text style={{ flex: 1, color: ColorScheme.labelColor }}>
            {props.label}
          </Text>
          <Text style={{ marginLeft: "auto", color: ColorScheme.text }}>
            {label()}
          </Text>
        </View>
      </Pressable>
      <Modal
        visible={open}
        animationType="slide"
        transparent={true}
        onRequestClose={() => {
          setOpen(!open);
        }}
      >
        <ScrollView
          style={{
            flex: 1,
            backgroundColor: ColorScheme.background,
            padding: 10,
            zIndex: 0,
          }}
        >
          {renderHeader()}
          {renderWeekdays()}
          <View style={{ flex: 4, flexDirection: "column" }}>
            {renderDays()}
          </View>
          <View style={{ paddingTop: 5, flexDirection: "row", flex: 1 }}>
            <View style={{ paddingTop: 5, flex: 1 }}>
              <Text style={{ fontSize: 22, color: ColorScheme.text }}>
                Hour:
              </Text>
              <Input
                maxLength={2}
                value={hourEntry}
                defaultValue={moment(selectedDateTime).format("hh")}
                onChangeText={(hour) => {
                  setHourEntry(hour);
                  setSelectedDateTime(
                    moment(selectedDateTime).hour(Number(hour))
                  );
                }}
                style={{
                  color: "white",
                  fontSize: 36,
                  borderBottomColor: ColorScheme.text,
                  borderBottomWidth: StyleSheet.hairlineWidth,
                }}
                keyboardType="numeric"
              />
            </View>
            <View style={{ paddingTop: 5, flex: 1 }}>
              <Text style={{ fontSize: 22, color: ColorScheme.text }}>
                Minute:
              </Text>
              <Input
                maxLength={2}
                value={minuteEntry}
                defaultValue={moment(selectedDateTime).format("mm")}
                onChangeText={(minute) => {
                  setMinuteEntry(minute);
                  setSelectedDateTime(
                    moment(selectedDateTime).minute(Number(minute))
                  );
                }}
                style={{
                  color: "white",
                  fontSize: 36,
                  borderBottomColor: ColorScheme.text,
                  borderBottomWidth: StyleSheet.hairlineWidth,
                }}
                keyboardType="numeric"
              />
            </View>
          </View>
          <View style={{ flex: 2, marginTop: 20 }}>
            <Text
              style={{
                color: ColorScheme.text,
                textAlign: "center",
                fontSize: 18,
              }}
            >
              Selected: {selectedDateTime.format("ddd, MMM DD, YYYY HH:mm")}
            </Text>
          </View>
          <View style={{ flexDirection: "row", flex: 1, marginTop: 20 }}>
            <View style={{ padding: 5, zIndex: 0, flex: 1 }}>
              <Button
                onPress={() => {
                  props.onDateTimeSelect(selectedDateTime.toDate());
                  setOpen(!open);
                }}
                title="Select"
              />
            </View>
            <View style={{ padding: 5, zIndex: 0, flex: 1 }}>
              <Button
                onPress={() => {
                  let today = moment(new Date());
                  today.hour(12);
                  today.minute(0);
                  setHourEntry("12");
                  setMinuteEntry("00");
                  setDate(today.toDate());
                  setSelectedDateTime(today);
                }}
                title="Today"
              />
            </View>
            <View style={{ padding: 5, zIndex: 0, flex: 1 }}>
              <Button
                onPress={() => {
                  setOpen(!open);
                }}
                title="Cancel"
              />
            </View>
          </View>
        </ScrollView>
      </Modal>
    </View>
  );
};
```
