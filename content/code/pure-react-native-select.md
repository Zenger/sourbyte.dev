+++
title = "Pure React Native Select"
date = "2023-07-02 03:38:00"
+++

I was frustated with the amount of dependencies you have to pull just to get a simple picker/selector in react native.
So I wrote my own component thats really simple.
It uses a modal to render select list, it can accept single or multiple entries, doesnt depend on anything. I used simple utf-8 arrow and dot to denote selection but they can be replaced with icons if you wish to pull in a dependency like [react-native-vector-icons](https://www.npmjs.com/package/react-native-vector-icons).

In essence it works like this:

<video controls>
  <source src="/assets/videos/pure-react-native-select.mp4" type="video/mp4">
</video>

the api is really easy and intuitive:

```tsx
<Select
  label="Makes"
  items={Makes.map((item, index) => {
    return { label: item, value: index };
  })}
  value={myValue}
  onChange={(value) => {
    console.log(value);
  }}
/>
```

and the source code, there are a couple constants at the top for styling but it should be easy enough to adjust.

```tsx
import { useEffect, useState } from "react";
import {
  Modal,
  Pressable,
  ScrollView,
  StyleSheet,
  Text,
  View,
} from "react-native";

const color_primary = "#3366FF";
const color_text = "#000000";
const color_background = "white";

interface SelectItem {
  label: string;
  value: any;
}

interface _SelectItem extends SelectItem {
  selected: boolean;
}

interface SelectProps {
  label: string;
  items: SelectItem[];
  multiple?: boolean;
  value?: any;
  onChange?: (value: any) => void;
}

export const Select: React.FC<SelectProps> = (props: SelectProps) => {
  const [items, setItems] = useState<SelectItem[]>(props.items || []);
  const [_items, _setItems] = useState<_SelectItem[]>([]);
  const [open, setOpen] = useState<boolean>(false);

  useEffect(() => {
    setItems(props.items);
    let selectedValues = props.value;
    if (!Array.isArray(props.value)) {
      selectedValues = [props.value];
    }

    if (Array.isArray(props.items)) {
      let i = props.items.map((item) => {
        return {
          label: item.label,
          value: item.value,
          selected: selectedValues.includes(item.value),
        };
      });
      _setItems(i);
    }
  }, [props.value]);

  const setItemSelected = (item: SelectItem) => {
    if (props.multiple) {
      let _i = _items.map((_item) => {
        if (_item.value === item.value) {
          _item.selected = !_item.selected;
        }
        return _item;
      });
      _setItems(_i);
      let selectedValues = _i
        .filter((item) => {
          return item.selected;
        })
        .map((item) => {
          return item.value;
        });
      if (props.onChange) {
        props.onChange(selectedValues);
        if (!props.multiple) {
          setOpen(false);
        }
      }
    } else {
      let _i = _items.map((_item) => {
        if (_item.value === item.value) {
          _item.selected = true;
        } else {
          _item.selected = false;
        }
        return _item;
      });
      _setItems(_i);
      if (props.onChange) {
        props.onChange(item.value);
        if (!props.multiple) {
          setOpen(false);
        }
      }
    }
  };

  const flexSize = () => {
    if (props.items.length < 6) {
      return 1;
    } else if (props.items.length < 11) {
      return 2;
    } else {
      return 10;
    }
  };

  const label = () => {
    return _items
      .filter((item) => {
        return item.selected;
      })
      .map((item) => {
        return item.label;
      })
      .join(", ");
  };

  const Element = (item: _SelectItem) => {
    return (
      <Pressable
        key={item.value}
        onPress={() => {
          setItemSelected(item);
        }}
        style={{
          ...{
            minHeight: 40,
            borderWidth: StyleSheet.hairlineWidth,
            borderColor: "black",
          },
          ...({ pressed }) => [
            pressed ? { opacity: 0.6, backgroundColor: "rgba(0,0,0,0.2)" } : {},
          ],
        }}
      >
        <View style={{ flex: 1, flexDirection: "row", flexWrap: "wrap" }}>
          <View style={{ flex: 1 }}>
            {!props.multiple && item.selected && (
              <Text
                style={{
                  marginTop: -15,
                  fontWeight: "bold",
                  marginLeft: 10,
                  fontSize: 50,
                }}
              >
                ·
              </Text>
            )}
          </View>
          <Text style={{ flex: 1, textAlign: "center", marginTop: 10 }}>
            {item.label}
          </Text>
          <View style={{ flex: 1 }}>
            {props.multiple && item.selected && (
              <Text style={{ marginTop: 10 }}>✓</Text>
            )}
          </View>
        </View>
      </Pressable>
    );
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
            borderWidth: 0.5,
            borderColor: "white",
            paddingHorizontal: 10,
            paddingVertical: 10,
            flexDirection: "row",
            backgroundColor: color_background,
          }}
        >
          <View style={{ flex: 1 }}>
            <Text style={{ color: color_primary }}>{props.label}</Text>
          </View>
          <View style={{ flex: 3 }}>
            <Text
              numberOfLines={1}
              style={{ textAlign: "right", color: color_text }}
            >
              {label()}
            </Text>
          </View>
        </View>
      </Pressable>

      <Modal animationType="slide" transparent={true} visible={open}>
        <View style={{ flex: 1 }}>
          <Pressable
            style={{ flex: 1, backgroundColor: "rgba(0,0,0,0.5)" }}
            onPress={() => {
              setOpen(!open);
            }}
          />
          <View style={{ flex: flexSize(), flexDirection: "row" }}>
            <Pressable
              style={{ flex: 1, backgroundColor: "rgba(0,0,0,0.5)" }}
              onPress={() => {
                setOpen(!open);
              }}
            />
            <View style={{ flex: 10, backgroundColor: "white" }}>
              <ScrollView>
                {_items.map((item) => {
                  return Element(item);
                })}
              </ScrollView>
            </View>
            <Pressable
              style={{ flex: 1, backgroundColor: "rgba(0,0,0,0.5)" }}
              onPress={() => {
                setOpen(!open);
              }}
            />
          </View>
          <Pressable
            style={{ flex: 1, backgroundColor: "rgba(0,0,0,0.5)" }}
            onPress={() => {
              setOpen(!open);
            }}
          />
        </View>
      </Modal>
    </View>
  );
};
```
