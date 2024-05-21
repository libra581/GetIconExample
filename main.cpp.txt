#include <cstdio>
#include <iostream>

#include <gio/gio.h>
#include <gtk/gtk.h>

#include <QBoxLayout>
#include <QIcon>
#include <QLabel>
#include <QLineEdit>
#include <QObject>
#include <QPushButton>
#include <QSize>
#include <QString>
#include <QtWidgets>

int main(int argc, char* argv[]) {
  gtk_init(&argc, &argv);

  QApplication app(argc, argv);
  QWidget root_widget;
  root_widget.resize(320, 240);
  QVBoxLayout* layout = new QVBoxLayout();
  QLabel* label_for_icon = nullptr;
  {
    label_for_icon = new QLabel();
    layout->addWidget(label_for_icon);
  }
  QLineEdit* line_edit = nullptr;
  {
    QHBoxLayout* h_layout = new QHBoxLayout();
    QLabel* label = new QLabel();
    label->setText("Расширение файла");
    line_edit = new QLineEdit();
    line_edit->setText("*.sh");
    h_layout->addWidget(label);
    h_layout->addWidget(line_edit);
    layout->addLayout(h_layout);
  }
  QPushButton* push_button = nullptr;
  {
    push_button = new QPushButton();
    push_button->setText("Подгрузить иконку");
    layout->addWidget(push_button);
  }
  root_widget.setLayout(layout);
  root_widget.show();
  root_widget.setWindowTitle("Get icon example");

  QObject::connect(push_button, &QPushButton::clicked, [&]() {
    const char* file_extension = line_edit->text().toUtf8();
    gboolean is_certain = FALSE;
    char* content_type = g_content_type_guess(file_extension, NULL, 0, &is_certain);
    if (content_type != NULL) {
      GAppInfo* app_info = g_app_info_get_default_for_type(content_type, FALSE);
      g_assert_nonnull(app_info);
      GIcon* icon = g_app_info_get_icon(app_info);
      if (G_IS_THEMED_ICON(icon) == true) {
        GThemedIcon* themed_icon = G_THEMED_ICON(icon);
        g_assert_nonnull(themed_icon);
        const gchar* const* icon_name = g_themed_icon_get_names(themed_icon);
        GtkIconTheme* theme = gtk_icon_theme_get_default();
        if (theme == NULL) {
          theme = gtk_icon_theme_new();
        }
        GtkIconInfo* gtk_icon = gtk_icon_theme_lookup_icon(theme, *icon_name, 48, GTK_ICON_LOOKUP_GENERIC_FALLBACK);
        const char* file_name = gtk_icon_info_get_filename(gtk_icon);
        QIcon qt_icon(QString::fromUtf8(file_name));
        if (qt_icon.isNull() == false) {
          label_for_icon->setPixmap(qt_icon.pixmap(QSize(48, 48)));
        }
        g_object_unref(gtk_icon);
      }
      g_object_unref(icon);
    }
  });

  return app.exec();
}