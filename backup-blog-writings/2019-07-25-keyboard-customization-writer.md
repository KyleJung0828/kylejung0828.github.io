---
layout: post
title:  "[Dev] Keyboard Customizations Writer"
date:   2019-07-25
tags: [dev]
---

```cpp
void CustomizationsWriter::writeKeymapElement(XMLStream* stream,
                                              const std::pair<Accelerator, CommandEnum>& acceleratorPair,
                                              OfBool bIsMasking)
{
    //  KH_J 2018.10.16 : KeyMap Element 저장은 나중에 호환 시 재분석 및 재구현 해야 한다.
    // 현재는 fciName (Command명)을 int로, kcmPrimary (KeyCode)를 여러 attribute로 나눠서 임시 구현한 것임.
    // kcmPrimary 및 kcmSecondary는 ST_ShortHexNumber 형식이어야 함.
    // 예를 들어 Shift+Ctrl+Alt+C는 0743으로 기록됨.

    XMLElement wKeymap("wne:keymap", stream);

    const Accelerator& accelerator = acceleratorPair.first;
    CommandEnum commandEnum = acceleratorPair.second;

    // Default/ModDefault Keymap에서 제거된 key는 "mask"라는 attribute로 표현함.
    if (bIsMasking)
    {
        // Write Masking status if erased default map element.
        wKeymap.attr("wne:mask", "1");
    }

    // Write Modifier
    OfString isCtrlDown = accelerator.isCtrlDown() ? "1" : "0";
    OfString isShiftDown = accelerator.isShiftDown() ? "1" : "0";
    OfString isAltDown = accelerator.isAltDown() ? "1" : "0";

    OfString bitStr = isAltDown + isCtrlDown + isShiftDown;

    std::bitset<3> modifierBitSet(bitStr); // 2^0 = shift, 2^1 = Ctrl, 2^2 = Alt (possibly 2^3 = cmd, but excluded in this case)

    OfString modifierStr = "0" + std::to_string(modifierBitSet.to_ulong());
    OF_DLOG(INFO) << "modifierStr is : " << modifierStr;

    // Write KeyCode
    auto vKeyCode = accelerator.getKeyCode(); // Note KH_J : ui::KeyboardCode is of uInt8 type.
    OF_DLOG(INFO) << "vKeyCode is : " << std::hex << std::showbase << vKeyCode; // display as hexadecimal and show base.

    std::stringstream ss;
    ss << std::hex << vKeyCode; // Note: Do not need to record base.
    OfString vKeyCodeStr = ss.str();

    vKeyCodeStr = modifierStr + vKeyCodeStr; // Combined to represent MS-style KeyCode.
    OF_DLOG(INFO) << "vKeyCodeStr is : " << vKeyCodeStr;

    wKeymap.attr("wne:kcmPrimary", vKeyCodeStr);

    // Write command Enum if additionally defined custom map element.
    OF_DLOG(INFO) << "commandEnum as int : " << (int)(commandEnum);
    OfString commandEnumStr = getString(commandEnum);
    OF_DLOG(INFO) << "commandEnum as string : " << commandEnumStr;
    if (!bIsMasking)
    {
        XMLElement wFCI("wne:fci", stream);
        wFCI.attr("wne:fciName", commandEnumStr);
        wFCI.attr("wne:swArg", "0000");
    }

    /* Parser Test (Command Enum) */
    const char* commandEnumChar = commandEnumStr.c_str();
    CommandEnum commandEnum_converted = getCommandEnumValue(commandEnumChar);

    OF_DLOG(INFO) << "string to commandEnum : " << (int)commandEnum_converted;
    /* ----------- End Parser Test -----------*/
}
```
