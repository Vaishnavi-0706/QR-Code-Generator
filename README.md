# QR-Code-Generator


import qrcode
<br>
import PySimpleGUI as sg
<br>
from PIL import Image
<br>
import io
<br>
import os
<br>
from tkinter import colorchooser
<br>

def create_qr_code(text, qr_color, back_color="white", logo_path=None):
<br>
    qr = qrcode.QRCode(
    <br>
        version=1,
        <br>
        error_correction=qrcode.constants.ERROR_CORRECT_H,
        box_size=10,
        <br>
        border=4,
        <br>
    )
    <br>
    qr.add_data(text)
    <br>
    qr.make(fit=True)
    <br>

    img = qr.make_image(fill_color=qr_color, back_color=back_color).convert("RGBA")
<br>
    if logo_path and os.path.isfile(logo_path):
    <br>
        logo = Image.open(logo_path).convert("RGBA")
        <br>
        logo_width, logo_height = logo.size
        <br>
        img_width, img_height = img.size
        <br>
        scale_factor = min(img_width // 3, img_height // 3, logo_width, logo_height)
        <br>
        logo.thumbnail((scale_factor, scale_factor))
        <br>
        logo_pos = ((img_width - logo.width) // 2, (img_height - logo.height) // 2)
<br>
        logo_no_alpha = Image.new("RGB", logo.size, (255, 255, 255))
        <br>
        logo_no_alpha.paste(logo, mask=logo.split()[3])
<br>
        img.paste(logo_no_alpha, logo_pos, mask=logo.split()[3])
<br>
    return img
<br>

def convert_to_bytes(img):
<br>
    with io.BytesIO() as bio:
    <br>
        img.save(bio, format="PNG")
        <br>
        return bio.getvalue()
        <br>

layout = [
<br>
    [sg.Text("Enter the text to encode in QR code:")],
    <br>
    [sg.InputText(key="INPUT_TEXT")],
    <br>
    [sg.Text("QR code color: "), sg.Button("Pick color", key="PICK_COLOR"), sg.Text("black", key="QR_COLOR", size=(10, 1))],
    <br>
    [sg.Text("Optional: Select a logo image to embed in QR code:")],
    <br>
    [sg.InputText(key="LOGO_PATH"), sg.FileBrowse(file_types=(("Image Files", "*.png;*.jpg;*.jpeg;*.bmp;*.gif"),))],
    <br>
    [sg.Button("Submit"), sg.Button("Exit")],
    <br>
    [sg.Image(key="QR_IMAGE")],
    <br>
    [sg.Button("Save"), sg.Text("QR code image not saved", key="SAVE_STATUS", text_color="red", visible=False)]
    <br>
]
<br>
window = sg.Window("QR Code Generator", layout)
<br>
chosen_color = 'black'
<br>
while True:
<br>
    event, values = window.read()
    <br>
    if event == sg.WIN_CLOSED or event == "Exit":
    <br>
        break
        <br>

    if event == "PICK_COLOR":
    <br>
        chosen_color = colorchooser.askcolor()[1]
        <br>
        if chosen_color:
        <br>
            window["QR_COLOR"].update(chosen_color)
<br>
    if event == "Submit":
    <br>
        text = values["INPUT_TEXT"]
        <br>
        qr_color = chosen_color
        <br>
        logo_path = values["LOGO_PATH"]
        <br>
        img = create_qr_code(text, qr_color=qr_color, logo_path=logo_path)
        <br>
        img_byte_array = convert_to_bytes(img)
        <br>
        window["QR_IMAGE"].update(data=img_byte_array)
        <br>
        window["SAVE_STATUS"].update(visible=False)
        <br>
    if event == "Save":
    <br>
        if "img" in locals():
        <br>
            file_name = sg.popup_get_file("Save QR code image as", save_as=True, file_types=(("PNG Files", "*.png"),))
            if file_name:
            <br>
                img.save(f'{file_name}.png')
                <br>
                window["SAVE_STATUS"].update(f"QR code image saved as \n{file_name}.png", text_color="black", visible=True)
                <br>
            else:
            <br>
                window["SAVE_STATUS"].update("QR code image not saved", text_color="red", visible=True)
                <br>
        else:
        <br>
            sg.popup("No QR code to save. Please generate a QR code first.")
            <br>

window.close()

