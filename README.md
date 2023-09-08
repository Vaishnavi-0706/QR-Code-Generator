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
        logo = Image.open(logo_path).convert("RGBA")
        logo_width, logo_height = logo.size
        img_width, img_height = img.size
        scale_factor = min(img_width // 3, img_height // 3, logo_width, logo_height)
        logo.thumbnail((scale_factor, scale_factor))
        logo_pos = ((img_width - logo.width) // 2, (img_height - logo.height) // 2)

        logo_no_alpha = Image.new("RGB", logo.size, (255, 255, 255))
        logo_no_alpha.paste(logo, mask=logo.split()[3])

        img.paste(logo_no_alpha, logo_pos, mask=logo.split()[3])

    return img

def convert_to_bytes(img):
    with io.BytesIO() as bio:
        img.save(bio, format="PNG")
        return bio.getvalue()

layout = [
    [sg.Text("Enter the text to encode in QR code:")],
    [sg.InputText(key="INPUT_TEXT")],
    [sg.Text("QR code color: "), sg.Button("Pick color", key="PICK_COLOR"), sg.Text("black", key="QR_COLOR", size=(10, 1))],
    [sg.Text("Optional: Select a logo image to embed in QR code:")],
    [sg.InputText(key="LOGO_PATH"), sg.FileBrowse(file_types=(("Image Files", "*.png;*.jpg;*.jpeg;*.bmp;*.gif"),))],
    [sg.Button("Submit"), sg.Button("Exit")],
    [sg.Image(key="QR_IMAGE")],
    [sg.Button("Save"), sg.Text("QR code image not saved", key="SAVE_STATUS", text_color="red", visible=False)]
]

window = sg.Window("QR Code Generator", layout)
chosen_color = 'black'

while True:
    event, values = window.read()
    if event == sg.WIN_CLOSED or event == "Exit":
        break

    if event == "PICK_COLOR":
        chosen_color = colorchooser.askcolor()[1]
        if chosen_color:
            window["QR_COLOR"].update(chosen_color)

    if event == "Submit":
        text = values["INPUT_TEXT"]
        qr_color = chosen_color
        logo_path = values["LOGO_PATH"]
        img = create_qr_code(text, qr_color=qr_color, logo_path=logo_path)
        img_byte_array = convert_to_bytes(img)
        window["QR_IMAGE"].update(data=img_byte_array)
        window["SAVE_STATUS"].update(visible=False)

    if event == "Save":
        if "img" in locals():
            file_name = sg.popup_get_file("Save QR code image as", save_as=True, file_types=(("PNG Files", "*.png"),))
            if file_name:
                img.save(f'{file_name}.png')
                window["SAVE_STATUS"].update(f"QR code image saved as \n{file_name}.png", text_color="black", visible=True)
            else:
                window["SAVE_STATUS"].update("QR code image not saved", text_color="red", visible=True)
        else:
            sg.popup("No QR code to save. Please generate a QR code first.")

window.close()

