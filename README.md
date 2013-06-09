Wines
=====
import org.powerbot.core.event.events.MessageEvent;
import org.powerbot.core.event.listeners.MessageListener;
import org.powerbot.core.event.listeners.PaintListener;
import org.powerbot.core.script.ActiveScript;
import org.powerbot.core.script.job.state.Node;
import org.powerbot.core.script.job.state.Tree;
import org.powerbot.game.api.Manifest;
import org.powerbot.game.api.methods.Calculations;
import org.powerbot.game.api.methods.Tabs;
import org.powerbot.game.api.methods.Walking;
import org.powerbot.game.api.methods.Widgets;
import org.powerbot.game.api.methods.input.Mouse;
import org.powerbot.game.api.methods.interactive.Players;
import org.powerbot.game.api.methods.node.GroundItems;
import org.powerbot.game.api.methods.node.SceneEntities;
import org.powerbot.game.api.methods.tab.Inventory;
import org.powerbot.game.api.methods.tab.Magic;
import org.powerbot.game.api.methods.tab.Skills;
import org.powerbot.game.api.methods.widget.Bank;
import org.powerbot.game.api.methods.widget.Camera;
import org.powerbot.game.api.util.Random;
import org.powerbot.game.api.util.Time;
import org.powerbot.game.api.util.Timer;
import org.powerbot.game.api.wrappers.Entity;
import org.powerbot.game.api.wrappers.Locatable;
import org.powerbot.game.api.wrappers.Tile;
import org.powerbot.game.api.wrappers.node.GroundItem;
import org.powerbot.game.api.wrappers.node.SceneObject;
import org.powerbot.game.api.wrappers.widget.WidgetChild;
 
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.net.URL;
import java.net.URLConnection;
 
/**
 * Created with IntelliJ IDEA. User: User Date: 4/5/13 Time: 11:06 AM Type: RSBot
 * Script
 */
@Manifest(name = "Wine Grabber ", description = "Start at Falador bank", authors = "Person", version = 1.1)
public class Graber extends ActiveScript implements PaintListener,
                MessageListener {
 
        Tile[] TEMPLE_PATH = new Tile[] { new Tile(2944, 3371, 0),
                        new Tile(2946, 3376, 0), new Tile(2950, 3379, 0),
                        new Tile(2955, 3379, 0), new Tile(2960, 3381, 0),
                        new Tile(2965, 3382, 0), new Tile(2966, 3387, 0),
                        new Tile(2966, 3392, 0), new Tile(2966, 3397, 0),
                        new Tile(2966, 3402, 0), new Tile(2965, 3407, 0),
                        new Tile(2964, 3412, 0), new Tile(2961, 3416, 0),
                        new Tile(2956, 3418, 0), new Tile(2952, 3421, 0),
                        new Tile(2951, 3426, 0), new Tile(2949, 3431, 0),
                        new Tile(2947, 3436, 0), new Tile(2946, 3441, 0),
                        new Tile(2946, 3446, 0), new Tile(2948, 3456, 0),
                        new Tile(2948, 3461, 0), new Tile(2952, 3467, 0),
                        new Tile(2952, 3475, 0) };
 
        private final int wineID = 245;
        private final int tableID = 78230;
        private final int lawRune = 563;
        private final int waterRune = 555;
        private int telekGrabIndex = -1;
        private int faladorTeleIndex = -1;
        private int magicXpStarted;
        private int xpGained;
        private int xpPerHour;
        private int totalWineGained = 0;
        private int totalWineMissed = 0;
        private int profit = 0;
        private int profitPH = 0;
        private int price;
 
        private boolean useActionBar = false;
        private boolean guiVisible = true;
 
        private double version = getClass().getAnnotation(Manifest.class).version();
 
        final Timer timer = new Timer(0);
 
        public enum LocationState {
                AT_WINES, AT_BANK, IN_THE_WAY
        }
 
        public enum State {
                GRAB, TELEPORT, BANK, WALK_TO_TEMPLE, WALK_TO_BANK
        }
 
        private LocationState getCurrentLocation() {
                Entity bank = Bank.getNearest();
                SceneObject table = SceneEntities.getNearest(tableID);
 
                if (bank != null && Calculations.distanceTo((Locatable) bank) <= 7) {
                        return LocationState.AT_BANK;
                }
                if (table != null && table.isOnScreen()) {
                        return LocationState.AT_WINES;
                } else
                        return LocationState.IN_THE_WAY;
        }
 
        public State getAction() {
                final LocationState currentLoc = getCurrentLocation();
                if (isFull()) {
                        switch (currentLoc) {
                        case AT_BANK:
                                return State.BANK;
 
                        case AT_WINES:
                                return State.TELEPORT;
 
                        case IN_THE_WAY:
                                return State.WALK_TO_BANK;
 
                        }
 
                        return null;
                }
                if (!isFull()) {
                        switch (currentLoc) {
                        case AT_WINES:
                                return State.GRAB;
 
                        case IN_THE_WAY:
                        case AT_BANK:
                                if (Inventory.contains(lawRune)
                                                && Inventory.contains(waterRune)) {
                                        return State.WALK_TO_TEMPLE;
                                } else
                                        return State.BANK;
                        }
                }
 
                return null;
        }
 
        private final Tree scriptTree = new Tree(new Node[] { new MainLoop() });
        @Override
        public int loop() {
                final Node stateNode = scriptTree.state();
                if (stateNode != null) {
                        scriptTree.set(stateNode);
                        final Node setNode = scriptTree.get();
                        if (setNode != null) {
                               
                                getContainer().submit(setNode);
                                setNode.join();
                        }
                }
 
                return Random.nextInt(100, 200);
        }
 
        private class MainLoop extends Node {
 
                @Override
                public boolean activate() {
                        return !guiVisible;
                }
 
                @Override
                public void execute() {
                        final State state = getAction();
                        switch (state) {
                        case GRAB:
                                GroundItem WINE = GroundItems.getNearest(wineID);
                                SceneObject TABLE = SceneEntities.getNearest(tableID);
                                WidgetChild SKILLING_SPELLS = Widgets.get(275, 39);
                                WidgetChild TELEK_GRAB = Widgets.get(275, 16).getChild(32);
                                if (Players.getLocal().isIdle()) {
                                        if (!Magic.isSpellSelected()) {
                                                if (!useActionBar) {
                                                        if (Tabs.getCurrent().equals(Tabs.ABILITY_BOOK)) {
                                                                if (castSpell(SKILLING_SPELLS, TELEK_GRAB)) {
                                                                        Mouse.move(TABLE.getCentralPoint());
                                                                        sleep(350);
                                                                }
                                                        } else
                                                                Tabs.ABILITY_BOOK.open();
                                                } else {
                                                        if (telekGrabIndex != -1) {
                                                                TELEK_GRAB = Widgets.get(640, telekGrabIndex);
                                                                if (TELEK_GRAB.click(true)) {
                                                                        Mouse.move(TABLE.getCentralPoint());
                                                                        sleep(350);
                                                                }
                                                        } else {
                                                                getAndSet(640, 14332, "telek_hand");
                                                        }
                                                }
                                        }
                                        if (WINE != null && Magic.isSpellSelected()
                                                        && clickOnTable(WINE)) {
                                                sleep(500);
                                                Tabs.INVENTORY.open();
                                        }
                                }
                                break;
                        case TELEPORT:
                                WidgetChild FALADOR = Widgets.get(275, 16).getChild(34);
                                WidgetChild TELEPORT_SPELLS = Widgets.get(275, 38);
                               
                                if (!useActionBar) {
                                        Tabs.ABILITY_BOOK.open();
                                        if (castSpell(TELEPORT_SPELLS, FALADOR)) {
                                                sleep(3000);
                                                Tabs.INVENTORY.open();
                                        }
                                } else if (faladorTeleIndex != -1) {
                                        FALADOR = Widgets.get(640, faladorTeleIndex);
                                        if (FALADOR.click(true)) {
                                                sleep(3000);
                                                Tabs.INVENTORY.open();
                                        }
                                } else {
                                        getAndSet(640, 14337, "falador");
                                }
                                break;
                        case BANK:
                                Entity bank = Bank.getNearest();
                                if (Bank.isOpen()) {
                                        if (Inventory.getCount(wineID) != 0) {
                                                Bank.deposit(wineID, 0);
                                                sleep(500);
                                        }
                                        if (Inventory.getCount(lawRune) == 0) {
                                                Bank.withdraw(lawRune, 0);
                                                sleep(500);
                                        }
                                        if (Inventory.getCount(waterRune) == 0) {
                                                Bank.withdraw(waterRune, 0);
                                                sleep(500);
                                        }
 
                                        Bank.close();
                                } else if (bank != null) {
                                        Camera.turnTo((Locatable) bank);
                                        sleep(300);
                                        Bank.open();
                                }
 
                                break;
                        case WALK_TO_BANK:
                                Walking.newTilePath(TEMPLE_PATH).reverse().traverse();
                                if (!Walking.isRunEnabled() && Walking.getEnergy() >= 20) {
                                        Walking.setRun(true);
                                }
                                break;
                        case WALK_TO_TEMPLE:
                                Walking.newTilePath(TEMPLE_PATH).traverse();
                                if (!Walking.isRunEnabled() && Walking.getEnergy() >= 20) {
                                        Walking.setRun(true);
                                }
                                break;
                        }
                }
        }
 
        private boolean castSpell(final WidgetChild boxID, final WidgetChild spellID) {
                WidgetChild MAGIC_BOX = Widgets.get(275, 33);
 
                if (boxID.visible()) {
                        if (spellID.validate() && spellID.click(true)) {
                                return true;
                        }
                } else if (!isMagicBoxOpen() && MAGIC_BOX.validate()) {
                        MAGIC_BOX.click(true);
                } else
                        boxID.click(true);
 
                return false;
        }
 
        private boolean isFull() {
                WidgetChild lastItem = Widgets.get(679, 0).getChild(27);
                return (lastItem.getChildId() != -1);
        }
 
        private boolean isMagicBoxOpen() {
                WidgetChild SK = Widgets.get(275, 48);
                return SK.visible();
        }
 
        private void getAndSet(int widgetID, int textureID, String name) {
                for (WidgetChild x : Widgets.get(widgetID).getChildren()) {
                        if (x.getTextureId() == textureID) {
                                if (name.equals("falador")) {
                                        faladorTeleIndex = x.getIndex();
                                }
                                if (name.equals("telek_hand")) {
                                        telekGrabIndex = x.getIndex();
                                }
                                System.out.println(x.getIndex());
                        }
                }
        }
 
        private boolean clickOnTable(GroundItem gi) {
                Point p = getOnScreenPoint(gi);
                return gi != null && p != null && Mouse.click(p, true);
        }
 
        private Point getOnScreenPoint(GroundItem gi) {
                if (gi == null || !gi.validate() || !gi.isOnScreen())
                        return null;
                final int surfaceHeight = 500;
                SceneObject surface = SceneEntities.getAt(gi.getLocation());
                return (surface != null && surface.validate() && surface.getType() == SceneEntities.TYPE_INTERACTIVE) ? gi
                                .getLocation().getPoint(.5d, .5d, -surfaceHeight) : gi
                                .getCentralPoint();
        }
 
        /**
         * @Author <Coma>
         * */
        private static int getPrice(final int id) {
                try {
                        String price;
                        final URL url = new URL(
                                        "http://www.tip.it/runescape/json/ge_single_item?item="
                                                        + id);
                        final URLConnection con = url.openConnection();
                        con.setRequestProperty(
                                        "User-Agent",
                                        "Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10.4; en-US; rv:1.9.2.2) Gecko/20100316 Firefox/3.6.2");
                        final BufferedReader in = new BufferedReader(new InputStreamReader(
                                        con.getInputStream()));
                        String line;
                        while ((line = in.readLine()) != null) {
                                if (line.contains("mark_price")) {
                                        price = line.substring(line.indexOf("mark_price") + 13,
                                                        line.indexOf(",\"daily_gp") - 1);
                                        price = price.replace(",", "");
                                        in.close();
                                        return Integer.parseInt(price);
                                }
                        }
                } catch (final Exception ignored) {
                        return -1;
                }
                return -1;
        }
 
        private final Color color1 = new Color(0, 0, 0, 180);
        private final Color color2 = new Color(0, 0, 0);
 
        }
 
        @Override
        public void onStart() {
                Mouse.setSpeed(Mouse.Speed.VERY_FAST);
                magicXpStarted = Skills.getExperience(Skills.MAGIC);
                Gui gui = new Gui();
                gui.setVisible(true);
                price = getPrice(wineID);
        }
 
        @Override
        public void messageReceived(MessageEvent m) {
                if (m.getMessage().contains("Too late")) {
                        totalWineMissed++;
                }
        }
 
        public class Gui extends JFrame {
                public Gui() {
                        initComponents();
                }
 
                private void button1ActionPerformed(ActionEvent e) {
                        if (checkBox1.isSelected()) {
                                useActionBar = true;
                        }
                        setVisible(false);
                        guiVisible = false;
 
                }
 
                private void initComponents() {
 
                        label1 = new JLabel();
                        checkBox1 = new JCheckBox();
                        label2 = new JLabel();
                        button1 = new JButton();
 
                        // ======== this ========
                        setTitle("Wine Of Zamorak Grabber");
                        Container contentPane = getContentPane();
 
                        // ---- label1 ----
                        label1.setText("Grabs wine of zamorak");
                        label1.setForeground(Color.blue);
 
                        // ---- checkBox1 ----
                        checkBox1.setText("Use ActionBar");
 
                        // ---- label2 ----
                        label2.setText("Note:this is still under testing.");
 
                        // ---- button1 ----
                        button1.setText("Start");
                        button1.addActionListener(new ActionListener() {
                                @Override
                                public void actionPerformed(ActionEvent e) {
                                        button1ActionPerformed(e);
                                }
                        });
 
                        GroupLayout contentPaneLayout = new GroupLayout(contentPane);
                        contentPane.setLayout(contentPaneLayout);
                        contentPaneLayout
                                        .setHorizontalGroup(contentPaneLayout
                                                        .createParallelGroup()
                                                        .addGroup(
                                                                        contentPaneLayout
                                                                                        .createSequentialGroup()
                                                                                        .addGroup(
                                                                                                        contentPaneLayout
                                                                                                                        .createParallelGroup()
                                                                                                                        .addGroup(
                                                                                                                                        contentPaneLayout
                                                                                                                                                        .createSequentialGroup()
                                                                                                                                                        .addContainerGap()
                                                                                                                                                        .addComponent(
                                                                                                                                                                        label2))
                                                                                                                        .addGroup(
                                                                                                                                        contentPaneLayout
                                                                                                                                                        .createSequentialGroup()
                                                                                                                                                        .addContainerGap()
                                                                                                                                                        .addComponent(
                                                                                                                                                                        checkBox1))
                                                                                                                        .addGroup(
                                                                                                                                        contentPaneLayout
                                                                                                                                                        .createSequentialGroup()
                                                                                                                                                        .addGap(26,
                                                                                                                                                                        26,
                                                                                                                                                                        26)
                                                                                                                                                        .addComponent(
                                                                                                                                                                        label1))
                                                                                                                        .addGroup(
                                                                                                                                        contentPaneLayout
                                                                                                                                                        .createSequentialGroup()
                                                                                                                                                        .addGap(52,
                                                                                                                                                                        52,
                                                                                                                                                                        52)
                                                                                                                                                        .addComponent(
                                                                                                                                                                        button1)))
                                                                                        .addContainerGap(26,
                                                                                                        Short.MAX_VALUE)));
                        contentPaneLayout
                                        .setVerticalGroup(contentPaneLayout
                                                        .createParallelGroup()
                                                        .addGroup(
                                                                        contentPaneLayout
                                                                                        .createSequentialGroup()
                                                                                        .addContainerGap()
                                                                                        .addComponent(label1)
                                                                                        .addPreferredGap(
                                                                                                        LayoutStyle.ComponentPlacement.UNRELATED)
                                                                                        .addComponent(checkBox1)
                                                                                        .addGap(7, 7, 7)
                                                                                        .addComponent(label2)
                                                                                        .addGap(18, 18, 18)
                                                                                        .addComponent(button1)
                                                                                        .addContainerGap(12,
                                                                                                        Short.MAX_VALUE)));
                        setSize(195, 165);
                        setLocationRelativeTo(getOwner());
                        // JFormDesigner - End of component initialization
                        // //GEN-END:initComponents
                }
 
                // JFormDesigner - Variables declaration - DO NOT MODIFY
                // //GEN-BEGIN:variables
                // Generated using JFormDesigner Evaluation license - Boss isme
                private JLabel label1;
                private JCheckBox checkBox1;
                private JLabel label2;
                private JButton button1;
                // JFormDesigner - End of variables declaration //GEN-END:variables
        }
}
