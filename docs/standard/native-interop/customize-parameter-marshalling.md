---
title: Customizing parameter marshalling - .NET
description: Learn how to customize how .NET marshals your parameters to a native representation.
ms.date: 01/18/2019
ms.topic: how-to
---

# Customize parameter marshalling

When the .NET runtime's default parameter marshalling behavior doesn't do what you want, use can use the <xref:System.Runtime.InteropServices.MarshalAsAttribute?displayProperty=nameWithType> attribute to customize how your parameters are marshalled. These customization features do not apply when [runtime marshalling is disabled](disabled-marshalling.md).

> [!NOTE]
> Source-generated interop for [P/Invokes](./pinvoke-source-generation.md) and [COM](./comwrappers-source-generation.md) only respects a small subset of <xref:System.Runtime.InteropServices.MarshalAsAttribute> on parameters. It is recommended to use <xref:System.Runtime.InteropServices.Marshalling.MarshalUsingAttribute> for source-generated interop instead. For more information, see [Custom marshalling for source generation](./custom-marshalling-source-generation.md).

## Customizing string parameters

.NET has a variety of formats for marshalling strings. These methods are split into distinct sections on C-style strings and Windows-centric string formats.

### C-style strings

Each of these formats passes a null-terminated string to native code. They differ by the encoding of the native string.

| `System.Runtime.InteropServices.UnmanagedType` value | Encoding |
|------------------------------------------------------|----------|
| LPStr | ANSI |
| LPUTF8Str | UTF-8 |
| LPWStr | UTF-16 |
| LPTStr | UTF-16 |

The <xref:System.Runtime.InteropServices.UnmanagedType.VBByRefStr?displayProperty=nameWithType> format is slightly different. Like `LPWStr`, it marshals the string to a native C-style string encoded in UTF-16. However, the managed signature has you pass in the string by reference and the matching native signature takes the string by value. This distinction allows you to use a native API that takes a string by value and modifies it in-place without having to use a `StringBuilder`. We recommend against manually using this format since it's prone to cause confusion with the mismatching native and managed signatures.

### Windows-centric string formats

When interacting with COM or OLE interfaces, you'll likely find that the native functions take strings as `BSTR` arguments. You can use the <xref:System.Runtime.InteropServices.UnmanagedType.BStr?displayProperty=nameWithType> unmanaged type to marshal a string as a `BSTR`.

If you're interacting with WinRT APIs, you can use the <xref:System.Runtime.InteropServices.UnmanagedType.HString?displayProperty=nameWithType> format to marshal a string as an `HSTRING`.

## Customize array parameters

.NET also provides you multiple ways to marshal array parameters. If you're calling an API that takes a C-style array, use the <xref:System.Runtime.InteropServices.UnmanagedType.LPArray?displayProperty=nameWithType> unmanaged type. If the values in the array need customized marshalling, you can use the <xref:System.Runtime.InteropServices.MarshalAsAttribute.ArraySubType> field on the `[MarshalAs]` attribute for that.

If you're using COM APIs, you'll likely have to marshal your array parameters as `SAFEARRAY*`s. To do so, you can use the <xref:System.Runtime.InteropServices.UnmanagedType.SafeArray?displayProperty=nameWithType> unmanaged type. The default type of the elements of the `SAFEARRAY` can be seen in the table on [customizing `object` fields](customize-struct-marshalling.md#marshal-systemobject). You can use the <xref:System.Runtime.InteropServices.MarshalAsAttribute.SafeArraySubType?displayProperty=nameWithType> and <xref:System.Runtime.InteropServices.MarshalAsAttribute.SafeArrayUserDefinedSubType?displayProperty=nameWithType> fields to customize the exact element type of the `SAFEARRAY`.

## Customizing Boolean or decimal parameters

For information on marshalling Boolean or decimal parameters, see [Customizing structure marshalling](customize-struct-marshalling.md).

## Customizing object parameters (Windows-only)

On Windows, the .NET runtime provides a number of different ways to marshal object parameters to native code.

### Marshalling as specific COM interfaces

If your API takes a pointer to a COM object, you can use any of the following `UnmanagedType` formats on an `object`-typed parameter to tell .NET to marshal as these specific interfaces:

- `IUnknown`
- `IDispatch`
- `IInspectable`

Additionally, if your type is marked `[ComVisible(true)]` or you're marshalling the `object` type, you can use the <xref:System.Runtime.InteropServices.UnmanagedType.Interface?displayProperty=nameWithType> format to marshal your object as a COM callable wrapper for the COM view of your type.

### Marshalling to a `VARIANT`

If your native API takes a Win32 `VARIANT`, you can use the <xref:System.Runtime.InteropServices.UnmanagedType.Struct?displayProperty=nameWithType> format on your `object` parameter to marshal your objects as `VARIANT`s. See the documentation on [customizing `object` fields](customize-struct-marshalling.md#marshal-systemobject) for a mapping between .NET types and `VARIANT` types.

### Custom marshallers

If you want to project a native COM interface into a different managed type, you can use the `UnmanagedType.CustomMarshaler` format and an implementation of <xref:System.Runtime.InteropServices.ICustomMarshaler> to provide your own custom marshalling code.
